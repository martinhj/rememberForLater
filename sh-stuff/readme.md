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
