# rememberForLater

What I want to remember for later

## Self-overwriting memoized (smart) getters
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get#Smart_self-overwriting_lazy_getters
```javascript
get notifier() {
  delete this.notifier;
  return this.notifier = document.getElementById('bookmarked-notification-anchor');
},
```

## Use curly braces with switch
It is support for scopes in switch also
```javascript
//not:
(({ someProp }, otherVar) => {
switch (expression)
  case 'onething': 
    const { someProp } = otherVar     // Doing this yesterday made a lot of mess in the scope.
    return someProp                   // Babel did not throw even tho 'someProp' was in the
                                      // scope already.
}
)(someVar)
```

```javascript
//do:
(({ someProp }, otherVar) => {
switch (expression)
  case 'onething': {
    const { someProp } = otherVar           // Now it got it's own scope!
    return someProp   
  }
}
)(someVar)
```
