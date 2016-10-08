# JavaScript bind()

## Objectives

1. Explain how to use `Function.prototype.bind()`
2. Describe use cases for `bind()`
3. Describe pitfalls of using `bind()`

## What's `bind`?
![A bound kid](https://media.giphy.com/media/t2Tr9eyHjlB7i/giphy.gif)

`.bind()` is a method that creates a new function based on an existing function. Wait, what? That doesn't sound very useful. Well, the big thing about `.bind()` is that you can change the _function scope_ (i.e. `this`) for the newly created function. You can also add any arguments to the new function, allowing for partial application of functions.

## Use cases for bind

### Setting the scope
Let's say we're working on some kind of social media site, and we're storing our profile in an object:

```js
const myProfile = {
  id: 1382,
  firstName: 'Akuna',
  lastName: 'Matata',
  friends: [],
  getFriends() {
    fetch('http://api.example.com/1382/friends')
      .then(function (response) {
        return response.json();
      })
      .then(function (friends) {
        this.friends = friends;
      })
  }
}
```

When we run `myProfile.friends()`, it'll fetch a list from the API of our imaginary social media site, and set the `friends` array in the object to the result. Except... it won't. Not like this, anyway! The problem here is that we run into a scoping issue: the function callback in `.then()` has its own scope, so `myProfile.friends` will still be an empty array! A quick and dirty solution that you might see in the wild is 'caching' the right scope value in the outer function:


```js
getFriends() {
  const self = this;
  
  fetch('http://api.example.com/1382/friends')
    .then(function (response) {
      return response.json();
    })
    .then(function (friends) {
      self.friends = friends;
    });
}
```

By storing the right scope in a variable, we can access it when the function is done loading the friends, and they'll be set accordingly. But... that looks like a pretty ugly solution. In comes `.bind()`, the savior in our time of need!

```js
getFriends() {
  fetch('http://api.example.com/1382/friends')
    .then(function (response) {
      return response.json();
    })
    .then(function (friends) {
      this.friends = friends;
    }.bind(this));
}
```

Since functions in JS are also objects, we can call `.bind()` on it right away! This solves our scoping issue in the exact same way, except that this is much, _much_ cleaner.

As a sidenote, you'll see `.bind()` being used for scoping issues less and less, since ES2015 has introduced [arrow functions](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions). We can rewrite this in ES2015 using arrows functions like this:

```js
getFriends() {
  fetch('http://api.example.com/1382/friends')
    .then(response => response.json())
    .then(friends => this.friends = friends);
}
```

Wow, that looks much cleaner! Arrow functions will use the surrounding scope for `this` — they also have the benefit of having a shorthand form too (used above) to make our code a lot more terse. Sweet!

### Partial application
Let's say we have a function that multiplies two numbers:

```js
function multiplier(a, b) {
  return a * b;
}
```

We just noticed that we're doubling values a lot in our codebase. Rather than always typing `multiplier(2, x)`, we can use `.bind()` to _partially apply_ our function and create a new one with the first argument already filled in. That looks like this:

```js
const doubler = multiplier.bind(null, 2);

// Now we can use doubler like this:
console.log(doubler(5)); // prints 10
```

We're passing in `null` as the first argument because in this case, we don't care about changing the context of the function at all, since it's not being used anyway.


## Performance considerations
Now that we know what `.bind()` does behind the scenes, we need to take a moment to talk about its performance implications. Since `.bind()` creates a new function every time, it's important to not have this new function be created every time with the same context. The same thing goes for using arrow functions (as these are syntactical sugar that replace `.bind`).

For example, in React, a component can re-render many times. If a new function were created every time the component renders, there would be a slight performance impact. For a more in-depth look at why this is bad, take a look at [Never Bind in Render](https://ryanfunduk.com/articles/never-bind-in-render/). This is generally not noticable unless you're working with a super large application, but it's worth mentioning!

## Resources

- [MDN: Function.prototype.bind()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
- [MDN: Arrow functions](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [Never Bind in Render](https://ryanfunduk.com/articles/never-bind-in-render/)

<p class='util--hide'>View <a href='https://learn.co/lessons/javascript-bind'>Javascript bind()</a> on Learn.co and start learning to code for free.</p>
