---
layout: post
title: "Don't make Promises"
description: ""
categories: 
---

I've come to a conclusion that some may disagree with. That Promises, just aren't worth 
the trouble.

Of course by Promises, I mean JavaScript Promises. If you don't know what Promises are, 
you should read some of the many articles on it. Most of the articles that you will find
will probably be gushing about Promises like it's the Second Coming. And given that the 
problem that it's trying to solve is called "Callback _Hell_" it's not that surprising.

So here is the use case that Promises try to solve. Let's say that you have two or more 
asynchronous operations to do (probably AJAX calls). But you need to use the value 
generated by the first operation in the second, and the value from the second in the 
third, and so on. Most JavaScript Developers would (initialy) solve this using just 
nested callbacks like this.

``` JavaScript
op1(function(result1) {
  op2(result1,function(result2) {
    op3(result2, function(result3) {
      // and on and on and on...
    });
  });
});
```

It's ugly, hard to read, and it's called _Callback Hell_. However if each of our op 
functions returned a Promise we could instead write the code like this.

``` JavaScript
op1().then(function(result) {
  return op2(result);
}).then(function(result) {
  return op3(result);
})// .then... keep chaining until you are done.
```
