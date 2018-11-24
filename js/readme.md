# ES stuff I want to remember

## Array methods

### Object.assign to short and concise reduce key, value-array into object
```javascript
routes.reduce(
  (acc, [ key, value ]) => Object.assign({[key]: value}, acc),
  {}
)
```

### [].concat to easily flatten nested arrays

```javascript
routes = [
  [`/en-us`, [ `/`, `/route`, `/anotherRoute` ]]
  [`/ja-jp`, [ `/`, `/route`, `/anotherRoute` ]]
]
routes.reduce((acc, cur) => (acc.concat(...cur)), [])
```
