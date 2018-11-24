# ES stuff I want to remember

## Array methods

### Object.assign to short and concise reduce key, value-array into object
```javascript
const flattenRoutes = (routes) =>
  routes.reduce(
    (acc, cur) => Object.assign({ [`${cur[0].path}`]: cur[1] }, acc),
    {}
  )
```
