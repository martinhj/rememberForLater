# Reference arguments from last typed command
```bash
# the last argument
!$
# all the arguments
!:*
# the first argument
!:1
# arguments from first to third
!:1-3
# arguments from the second to the last one
!:2-$
# last command executed
!:0
```

## How I parallelized my web server test

Here I am hitting the web server with 1000 requests running in parallel. The `-P N` argument to xargs sets the number of parallel processes.
```bash
time printf "%s\0" {1..1000} | xargs -0 -I @ -P 1 curl -sSl http://localhost:5000/en-us > /dev/null
```

## Talking about server performance

I've been working on a corporate sales tool for some time.

Our app is a React app that delivers a server side rendered (SSR), prerendered, document at the time of the request and for further navigation on the site it will use client side rendering - fetching and render the content in the browser. This is often refered to as an isomorphic web app.

  > Isomorphic app
  > 
  > 

Together with our client we recently put it on a staging server in their hosting environment. We had agreed on putting it in Docker container with an Nginx reversed proxy in front of it.

Beforhand I knew we had a little performance problem in the implemented design that I wasn't happy about. All content is hosted on a headless CMS at the third party and the we do no caching to be sure to always be able to deliver the most recent content. Because of some technical limitations with the multi language support at the CMS provider all our rendered responses had a few requests to the CMS: Three per data object fetched from the server and each rendered page consisting of around three to four of these data objects.

From the usage of the site we could see that through a week the peek of pages rendered would be around 1500 requests per hour but a normal high would be 800-1000 page requests.

### Enter bad network throughput

The staging site went up, but running in their environment I got response time numbers a lot different from those we had had in our local dev environment (that isn't very suprising) but also a lot worse than on our online development server.

So I dug up good old `ping`:
```sh
ping remote-staging-server.somewhere
PING cae-xprp-rtp-vip.remote-staging-server.somewhere (404.040.404.040): 56 data bytes
64 bytes from 404.040.404.040: icmp_seq=0 ttl=52 time=203.134 ms
64 bytes from 404.040.404.040: icmp_seq=1 ttl=52 time=130.322 ms
Request timeout for icmp_seq 2
64 bytes from 404.040.404.040: icmp_seq=3 ttl=52 time=130.956 ms
64 bytes from 404.040.404.040: icmp_seq=4 ttl=52 time=138.056 ms
```
From my point of view, that is very, very bad. Compared to my favorite test host:
```sh
❯ ping 304.200.200.200
PING 304.200.200.200 (304.200.200.200): 56 data bytes
64 bytes from 304.200.200.200: icmp_seq=0 ttl=56 time=13.515 ms
64 bytes from 304.200.200.200: icmp_seq=1 ttl=56 time=6.011 ms
64 bytes from 304.200.200.200: icmp_seq=4 ttl=56 time=3.807 ms
64 bytes from 304.200.200.200: icmp_seq=0 ttl=56 time=4.775 ms
64 bytes from 304.200.200.200: icmp_seq=1 ttl=56 time=7.022 ms
64 bytes from 304.200.200.200: icmp_seq=2 ttl=56 time=8.384 ms
```
(This old DNS server, from the dawn of time, is still responding me in no time! I wonder if it is forgotten in some 1U somewhere and doesn't see much load. It is hosted by a renowned Internet service provider that isn't the same as the one I'm connected to at the present, but is the first one I manually had to configure and I use as a realistic best case scenario from time to time)

So our app is quite heavy on the requests and now this container is painting a picture of a landscape with quite scarce network resources.

The consquenses for the rendered page:

```md
200 GET	remote-staging-server.somewhere	en-us                           document  html  3854 ms
200 GET	remote-staging-server.somewhere	runtime-4ce3099b2e8a2c8e32c7.js script	  js    872 ms
200 GET	remote-staging-r.somewhere vendors~main-82ce81ae79c2a6e75900.js script	  js    566 ms
200 GET	remote-staging-server.somewhere	main-ef7e43e02464d048ea8f.js    script	  js    871 ms
```
_see the column to right_

The javascript files from the same server is for comparison. They are statically served files and the response time numbers isn't great it is still on another planet than the first row: The at request server rendered html document.

Compared to our staging server:
```md
200 GET sometestsite.herokuapp.com      en-us                                document  html	627 ms 
200 GET sometestsite.herokuapp.com	runtime-4ce3099b2e8a2c8e32c7.js      script    js  157 ms 
200 GET	sometestsite.herapp.com vendors~main-82ce81ae79c2a6e75900.js     script    js  415 ms 
200 GET	sometestsite.herokuapp.com	main-e0f4b241d7652c0e227d.js         script    js  157 ms
```

With the request load we could live with the latter numbers, but not what we are presented with now. Let's do some more testing with my little testing one liner - possible to set a number of parallel process to this (with `-P N` xargs argument):
```sh
time printf "%s\0" {1..50} | xargs -0 -I @ -P 1 time curl -sSl https://remote-staging-server.somewhere/en-us > /dev/n
ull
```


```sh
❯ time printf "%s\0" {1..50} | xargs -0 -I @ -P 1 time curl -sSl https://remote-staging-server.somewhere/en-us > /dev/n
ull
        5.20 real         0.02 user         0.00 sys
        4.07 real         0.01 user         0.00 sys
        1.21 real         0.01 user         0.00 sys
        1.03 real         0.01 user         0.00 sys
        0.87 real         0.01 user         0.00 sys
        1.33 real         0.01 user         0.00 sys
        0.92 real         0.01 user         0.00 sys
        1.01 real         0.01 user         0.00 sys
        1.08 real         0.01 user         0.00 sys
        1.06 real         0.01 user         0.00 sys
        1.30 real         0.01 user         0.00 sys
        1.04 real         0.01 user         0.00 sys
        0.97 real         0.01 user         0.00 sys
        1.17 real         0.01 user         0.00 sys
        0.82 real         0.01 user         0.00 sys
        0.92 real         0.01 user         0.00 sys
        1.14 real         0.02 user         0.00 sys
        1.54 real         0.01 user         0.00 sys
        1.28 real         0.01 user         0.00 sys
        1.18 real         0.01 user         0.00 sys
        0.95 real         0.01 user         0.00 sys
        1.03 real         0.01 user         0.00 sys
        1.03 real         0.01 user         0.00 sys
        1.24 real         0.01 user         0.00 sys
        0.86 real         0.02 user         0.00 sys
        0.90 real         0.02 user         0.00 sys
        0.89 real         0.01 user         0.00 sys
        1.09 real         0.01 user         0.00 sys
        1.28 real         0.01 user         0.00 sys
        1.59 real         0.01 user         0.00 sys
        1.22 real         0.01 user         0.00 sys
        0.90 real         0.01 user         0.00 sys
        0.99 real         0.02 user         0.00 sys
        1.05 real         0.01 user         0.00 sys
        0.91 real         0.01 user         0.00 sys
        1.00 real         0.01 user         0.00 sys
        1.26 real         0.02 user         0.00 sys
        0.91 real         0.01 user         0.00 sys
        0.97 real         0.01 user         0.00 sys
        1.02 real         0.01 user         0.00 sys
        0.89 real         0.02 user         0.00 sys
        1.15 real         0.01 user         0.00 sys
        1.25 real         0.01 user         0.00 sys
        1.12 real         0.02 user         0.00 sys
        1.32 real         0.01 user         0.00 sys
        0.93 real         0.01 user         0.00 sys
        1.11 real         0.01 user         0.00 sys
        0.81 real         0.01 user         0.00 sys
        0.86 real         0.01 user         0.00 sys
        1.05 real         0.01 user         0.00 sys
printf "%s\0" {1..50}  0.00s user 0.00s system 37% cpu 0.002 total
xargs -0 -I @ -P 1 time curl -sSl  > /dev/null  0.97s user 0.47s system 2% cpu 1:01.09 total
```

```sh
time printf "%s\0" {1..50} | xargs -0 -I @ -P 4 time curl -sSl https://remote-staging-server.somewhere/en-us > /dev/n
ull
```

```sh
        1.32 real         0.01 user         0.00 sys
        1.55 real         0.01 user         0.00 sys
        1.56 real         0.01 user         0.00 sys
        1.57 real         0.01 user         0.00 sys
        1.19 real         0.01 user         0.00 sys
        1.44 real         0.01 user         0.00 sys
        1.37 real         0.01 user         0.00 sys
        1.70 real         0.01 user         0.00 sys
        1.16 real         0.01 user         0.00 sys
        1.30 real         0.01 user         0.00 sys
        1.33 real         0.01 user         0.00 sys
        1.19 real         0.01 user         0.00 sys
        0.89 real         0.01 user         0.00 sys
        0.94 real         0.01 user         0.00 sys
        0.88 real         0.01 user         0.00 sys
        1.11 real         0.01 user         0.00 sys
        1.02 real         0.01 user         0.00 sys
        1.51 real         0.01 user         0.00 sys
        1.21 real         0.01 user         0.00 sys
        1.69 real         0.01 user         0.00 sys
        1.39 real         0.01 user         0.00 sys
        0.90 real         0.01 user         0.00 sys
        1.43 real         0.01 user         0.00 sys
        1.01 real         0.01 user         0.00 sys
        1.18 real         0.01 user         0.00 sys
        1.55 real         0.01 user         0.00 sys
        1.34 real         0.01 user         0.00 sys
        1.28 real         0.01 user         0.00 sys
        1.63 real         0.01 user         0.00 sys
        0.99 real         0.01 user         0.00 sys
        0.99 real         0.01 user         0.00 sys
        1.00 real         0.01 user         0.00 sys
        0.88 real         0.01 user         0.00 sys
        1.06 real         0.01 user         0.00 sys
        1.25 real         0.01 user         0.00 sys
        0.92 real         0.01 user         0.00 sys
        0.83 real         0.01 user         0.00 sys
        0.86 real         0.01 user         0.00 sys
        1.00 real         0.01 user         0.00 sys
        6.72 real         0.01 user         0.00 sys
        7.08 real         0.01 user         0.00 sys
        2.71 real         0.02 user         0.00 sys
        0.80 real         0.01 user         0.00 sys
        0.95 real         0.01 user         0.00 sys
        0.87 real         0.01 user         0.00 sys
        1.10 real         0.01 user         0.00 sys
        0.98 real         0.02 user         0.00 sys
        0.76 real         0.01 user         0.00 sys
        0.85 real         0.01 user         0.00 sys
        1.06 real         0.01 user         0.00 sys

xargs -0 -I @ -P 4 time curl -sSl  > /dev/null  0.93s user 0.47s system 7% cpu 18.214 total
```

Can we live with a worst case of 10 minute old version of the content, and in normal situations a version that most often is update within the last 10 seconds?

The rules:
Is the fetched data in cache older than ten minutes?
  YES:
  -> Is the data fetcher already been initialized and running:
    YES: ship the initialized data fetcher
    NO: init new fetch and return this fetcher, put it in cache
  
  NO:
  -> Is the data fetched within the last 5 seconds:
    Yes: -> Return the cached data and only this
    No:  -> Return cached data and start fetch (prefetch) the data and when complete put it in cache ready for next request.
