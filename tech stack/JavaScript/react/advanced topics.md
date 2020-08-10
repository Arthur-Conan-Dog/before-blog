# 性能

## Code-Splitting

将代码分割成多个 bundle，根据用户需要动态加载。

use dynamic ``import``. use React.lazy to lazy load components. use Suspense and ErrorBoundary to handle fallbacks.

Deciding where in your app to introduce code splitting can be a bit tricky. A good place to start is with routes. eg. use React.lazy & Suspense in an app routing by react-router.

React.lazy currently only supports default exports. => Need to reexport if the module uses the named export, eg. ``export const Component = ...`` => ``export { Component as default } from Components.js``

- [ ] code demo with React.lazy & Suspense

## Profiler API

The Profiler measures how often a React application renders and what the “cost” of rendering is. To opt into production profiling, React provides a special production build with profiling enabled.

```jsx
<Profiler id="component-to-track" onRender={callback}>
  <Component />
</Profiler>
```

[Interaction tracing with React](https://gist.github.com/bvaughn/8de925562903afd2e7a12554adcdda16)

# 控制数据流

## Context API

Context is designed to share data that can be considered “global” for a tree of React components, such as the current authenticated user, theme, or preferred language.

It lets us pass a value deep into the component tree without explicitly threading it through every component.

### Use case

Sometimes the same data needs to be accessible by many components in the tree, and at different nesting levels.

Before using Context, consider other ways such as components composition(inversion of control, open slots in top level component) and render props.

### Caveats

If a new object is always created for value, then every time Provider re-renders, all consumers will also re-render, and you cannot avoid this in ``shouldComponentUpdate``.

### FAQ

#### Change Context value dynamically

As for updating Context from a Nested Component, you can set the update function as part of the context value.

#### Consuming Multiple Contexts

Nested provider & Nested render functions.

- [ ] implement dark theme

## Refs

### Use Refs

Regular function or class components don’t receive the ref argument, and ref is not available in props either.

* Adding a Ref to a DOM Element

  React will assign the current property with the DOM element when the component mounts, and assign it back to null when it unmounts. ref updates happen before ``componentDidMount`` or ``componentDidUpdate`` lifecycle methods.

* Adding a Ref to a Class Component

  ``.current`` will refer to the class instance, and can be used to invoke instance methods.

* "Adding" a Ref to Function Components

  Normally, you may not use the ref attribute on function components because they don’t have instances. To be able to use it: forwardRef + useImperativeHandle(check hooks FAQ for more implementation details), or convert component to class.

* Exposing DOM Refs to Parent Components

  Use forwardRef, bind ref onto DOM inside child component.

* Callback Refs

  Instead of passing a ref attribute created by createRef(), you pass a function， which gives more fine-grain control over when refs are set and unset.

  Caveats: inline function => .bind(this) to avoid being called twice during updates. => TODO: why?

React.createRef vs useRef？

# 功能组件

## Fragments

## ErrorBoundry

To solve problem: a JavaScript error in a part of the UI shouldn’t break the whole app.

Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them.

Error boundaries **do not** catch errors for:

* Event handlers (learn more)

* Asynchronous code (e.g. setTimeout or requestAnimationFrame callbacks)

* Server side rendering

* Errors thrown in the error boundary itself (rather than its children)

The event handlers don’t happen during rendering.

## Portal

- [ ] implement a dialog with Portal

Event Bubbling Through Portals

Even though a portal can be anywhere in the DOM tree, it behaves like a normal React child in every other way. Features like context work exactly the same regardless of whether the child is a portal, as the portal still exists in the React tree regardless of position in the DOM tree.

# Lifecycles

http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/

# Diffing Algorithm(Reconciliation)

React = reconciler + renderer + etc. Renderer is pluggable, which can be used to adapt to different platforms.

## The Old One(Stack reconciler)

Use the system stack to keep track of computing changes. Updating the corresponding DOM node before it actually computes on the instance below this instance. Thus it may occupy the main thread for a long time and cause delays of UI rendering.

## Fiber

New reconciliation algorithm for React. Use cooperative scheduling.

Three types of object: React Element, Instance, DOM node.

fiber

```js
// fiber
{
  stateNode
  child
  sibling
  return
}
```

curent tree:

workInProgress tree: we don't want to make the changes to the dom while we're computing the changes.

phases

Phase 1: render / reconciliation

can be interrupted

requestIdleCallback

pending commit

Phase 2: commit

cannot be interrupted

switch pointer to the current tree and pointer to the workInProgress tree

double buffering

priorities

* Synchronous => same as stack rec.

* Task => before next tick

* Animation => before next frame

* High => pretty soon requestIdleCallback

* Low => minor delays ok requestIdleCallback

* OffScreen => prep for display/scroll requestIdleCallback

lifecycles happen in different phases

starvation problem: low priority tasks may never be executed

## draft structure

Fiber 是什么？

原来的算法有什么问题？=> 延伸：描述原来的算法面临的问题 以及当时采取的解决办法

Fiber 是如何改进的？主要思想是什么？

  递归 => 遍历 遍历过程中加入检查点从而跳出过长的执行过程；加入优先级合理化调度；为饥饿效应做特别处理。

# Virtual DOM

# Event Mechanism

## SyntheticEvent

A cross-browser wrapper around the browser’s native event. Use the ``nativeEvent`` attribute to get the underlying browser event.

## Event Pooling

The SyntheticEvent is pooled. This means that the SyntheticEvent object will be reused and all properties will be nullified after the event callback has been invoked.

Call ``event.persist()`` on the event to access the event properties in an asynchronous way, this will remove the synthetic event from the pool and allow references to the event to be retained by user code.

# Server-side rendering

https://medium.com/airbnb-engineering/isomorphic-javascript-the-future-of-web-apps-10882b7a2ebc#.4nyzv6jea

# Web components
