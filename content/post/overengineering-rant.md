---
title: "Rant: Overengineering"
date: 2019-07-16T22:48:10+05:00
draft: false
---

Consider a typical web dev situation: React, Redux, Material-UI, a new
JSS-decorated component that's going to be connected with Redux. We
create a new file, write the component code in it, then export it like
that:

```js
export default withStyles(styles)(connect(mapStateToProps)(MyComponent));
```

This simple line contains at least 5 functions in it. Let's tear this
down right-to-left:

+ `MyComponent` may possibly be a [functional component][ReactFunc]
(a function that accepts a single `props` argument and returns a React
element), so it might count as a function.
+ `mapStateToProps` is a [function that returns the mapping][mstp] of
the decorated component properties to Redux store state.
+ `connect(mapStateToProps)` will return...
+ ...a [decorator function][connect] that returns a wrapper component
around `MyComponent`. 

And we're only done with the first part of this abomination! Now let's
proceed to the outer block:

+ `styles` [has to be a function][styles] when you need access to theme
properties.
+ `withStyles(styles)` is a function that will return...
+ ...another wrapper component function.

Yep, that's how easy it is to overengineer in React/modern JS.
The code above might not exactly be a good example of a well-designed
component, but this gives you an idea of how things generally go.

[ReactFunc]: https://reactjs.org/docs/components-and-props.html
[mstp]: https://react-redux.js.org/using-react-redux/connect-mapstate
[connect]: https://react-redux.js.org/api/connect
[styles]: https://material-ui.com/styles/api/#withstyles-styles-options-higher-order-component
