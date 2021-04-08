# Event Loop

## What is event loop?

Spec: https://html.spec.whatwg.org/multipage/webappapis.html#event-loops

The JS engine doesn't run in isolation. It runs inside a hosting environment.

All these environments have a mechanism in them that handles executing multiple chunks of your program over time, at each moment invoking the JS engine. This mechanism is called the "event loop."

In other words, the JS engine has had no sense of time. It's the surrounding environment that has always scheduled "events" (JS code executions).

[You Don't Know JavaScript - Event Loop](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/async%20%26%20performance/ch1.md#event-loop)

## Event loop implemented by Browser(eg. Chrome)

### Recap JavaScript runtime

Simplified view of JavaScript runtime(eg. V8): heap(memory allocation) and call stack(execution contexts)

Things like setTimeout, DOM, HTTP request are not inside V8 source.

JavaScript = single thread runtime = single call stack = one piece of code one time

### How call stack works?

```jsx
function multiply(a, b) {
  return a * b;
}

function square(n) {
  return multiply(n, n);
}

function printSquare(n) {
  var squared = square(n);
  console.log(squared);
}

printSquare(4);
```

Actually, we have already seen visualization of call stack before:

```jsx
function foo() {
  throw new Error('oops');
}

function bar() {
  foo();
}

function baz() {
  bar();
}

baz();
```

and also, blowing stack:

```jsx
function foo() {
  return foo();
}

foo();
```

### What happens when things become slow?(eg. a huge while loop, network requests)

```jsx
var foo = $.getSyncByAjax('//foo.com');

console.log(foo);
```

_Note the call is synchronous._

This type of code will block the call stack, the browser can't do anything else. This is not ideal. The solution? asynchronous callbacks

### How does asynchronous callbacks work with call stack?

The JavaScript runtime can only do one thing a time, but the browser is more than just the runtime.

```jsx
console.log('Hi');

setTimeout(() => {
  console.log('there');
}, 5000);

console.log('JSConfEU');
```

setTimeout is an API provided by browser, it doesn't live in the V8 source. When it's being called, the browser will hold the callback and kick off a timer for us.

When the timer is done, browser will push the callback into the task queue(callback queue).

The event loop's job is to look at the stack and look at the task queue. It waits until the stack is empty, and takes the first thing in the queue and pushes it into the stack so that runtime could run it.

### How all of this interacts with rendering?

```jsx
[1, 2, 3, 4].forEach((i) => {
  console.log('processing sync');
  someSlowCode(); // => will block render
});

function asyncForEach(arr, cb) {
  arr.forEach((i) => setTimeout(() => cb(i), 0)); // => render will get a chance to run
}

asyncForEach([1, 2, 3, 4], (i) => {
  console.log('processing async');
  someSlowCode();
});
```

The browser would like to repaint the screen every 16.6ms, but it's constrained by what we're doing in JavaScript, it can't do a render if the call stack isn't clear.

The render call is kind like a callback itself.

However, the render queue is given a higher priority than the callbacks in callback queue(task queue). Each certain period of time, the browser queues a render into it.

While the runtime is handling with these slow synchronous code, the render is blocked, we cannot interact with DOMs, like the example in synchronous before.

For the asynchronous code, the render will get a chance to run.

[Visualizing the javascript runtime at runtime](https://github.com/latentflip/loupe)

## Futhermore: Microtask & Task(Macrotask)

From [processing model in html spec](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model), the basic algorithm can be summarized as:

1. Dequeue and run the oldest task from the macrotask queue (e.g. “script”).

2. Execute all microtasks, perform a microtask checkpoint, which may enqueue more microtasks.

3. Render changes if any.

4. If the macrotask queue is empty, wait till a macrotask appears.

5. Go to step 1.

One go-around of the event loop will have exactly one task being processed from the macrotask queue.

Refs that helps visualize this:

- [Event loop: microtasks and macrotasks](https://javascript.info/event-loop)

- [Difference between microtask and macrotask within an event loop context](https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)

- [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

- [What the heck is the event loop anyway? | Philip Roberts | JSConf EU](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

### Terms explanation

Tasks(Macrotasks): eg. script loads, setTimeout callbacks, event activates.

Tasks queue: Task queues are sets, not queues, their are many task queues. Step one of the event loop processing model grabs the first runnable task from the chosen queue, instead of dequeuing the first task.

Microtasks: eg. callbacks in promise.then, mutation observer callbacks, “under the cover” of `await`, scheduled by `queueMicrotask`.

Microtask queue: microtasks get processed after callbacks _as long as the call stack is empty_, and at the end of each task. Any additional microtasks queued during microtasks are added to the end of the queue and also processed. That’s important, as it guarantees that the application environment is basically the same (no mouse coordinate changes, no new network data, etc) between microtasks.

Renders: (requestAnimationFrame) styles layout painting

Animation callback queue: clear all items, and defer the ones that are queued while processing to the next frame

### Code sample 1

```jsx
console.log('script start');

setTimeout(function () {
  console.log('setTimeout');
}, 0);

Promise.resolve()
  .then(function () {
    console.log('promise1');
  })
  .then(function () {
    console.log('promise2');
  });

console.log('script end');
```

In what order should the logs appear?

It depends. Different browsers may show different results.

For Chrome is: script start, script end, promise1, promise2, setTimeout

After executing log 'script start' and log 'script end', the call stack is empty, and the microtasks queue has items, so 'promise1' gets executed. It enqueues another microtask 'promise2', which fullfills the microtasks queue again. Finally, task 'setTimeout' gets executed.

### Code sample 2

```js
button.addEventListener('click', () => {
  setTimeout(() => console.log('timeout 1'), 0);

  Promise.resolve().then(() => console.log('microtask 1'));

  console.log('listener 1');
});

button.addEventListener('click', () => {
  setTimeout(() => console.log('timeout 2'), 0);

  Promise.resolve().then(() => console.log('microtask 2'));

  console.log('listener 2');
});
```

scenario 1: click on the button. => dispatch click event(task), anonymous click handler 1 is pushed to the JS stack, push setimout's callback into task queue, push .then to microtask queue, log 'listener 1', stack is empty, dequeue microtask & log 'microtask 1', event bubbles(note that previous task is still going on), anonymous click handler 2 is pushed to the JS stack... At last, task of dispatching click event is done, setimeout callback 1 & 2 get to be executed.

scenario 2: directly call button.click(). => Since .click() will take a place in the call stack, before it's popped up from the call stack, all the microtasks won't be executed.

## Event loop implemented by Node.js

![](../static/images/nodejs-event-loop.png)
https://www.youtube.com/watch?v=PNa9OMajw9w
