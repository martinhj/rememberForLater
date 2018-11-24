# ES stuff I want to remember

## Array methods

### Object.assign to short and concise reduce key, value-array into object
```javascript
routes.reduce(
  (acc, [ key, value ]) => Object.assign({[key]: value}, acc),
  {}
)
```
