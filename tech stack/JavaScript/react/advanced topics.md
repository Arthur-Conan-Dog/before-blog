# 性能

## Profiler API

The Profiler measures how often a React application renders and what the “cost” of rendering is. To opt into production profiling, React provides a special production build with profiling enabled.

```jsx
<Profiler id="component-to-track" onRender={callback}>
  <Component />
</Profiler>
```

## Code-Splitting

将代码分割成多个 bundle，根据用户需要动态加载。

use dynamic ``import``. use React.lazy to lazy load components. use Suspense and ErrorBoundary to handle fallbacks.

Deciding where in your app to introduce code splitting can be a bit tricky. A good place to start is with routes. eg. use React.lazy & Suspense in an app routing by react-router.

React.lazy currently only supports default exports. => Need to reexport if the module uses the named export, eg. ``export const Component = ...`` => ``export { Component as default } from Components.js``

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

## Refs

### Use Refs

Regular function or class components don’t receive the ref argument, and ref is not available in props either.

- Adding a Ref to a DOM Element

  React will assign the current property with the DOM element when the component mounts, and assign it back to null when it unmounts. ref updates happen before ``componentDidMount`` or ``componentDidUpdate`` lifecycle methods.

- Adding a Ref to a Class Component

  ``.current`` will refer to the class instance, and can be used to invoke instance methods.

- "Adding" a Ref to Function Components

  Normally, you may not use the ref attribute on function components because they don’t have instances. To be able to use it: forwardRef + useImperativeHandle(check hooks FAQ for more implementation details), or convert component to class.

- Exposing DOM Refs to Parent Components

  Use forwardRef, bind ref onto DOM inside child component.

- Callback Refs

  Instead of passing a ref attribute created by createRef(), you pass a function， which gives more fine-grain control over when refs are set and unset.

# 功能组件

## Fragments

# Lifecycles

http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/

# Reconciliation

React = reconciler + renderer + etc. Renderer is pluggable, which can be used to adapt to different platforms.

## The diffing algorithm

When diffing two trees, React first compares the two root elements.

- elements of different types: whenever the root elements have different types, React will tear down the old tree and build the new tree from scratch. eg. Button => div, div => img etc.

- DOM elements of the same type: looks at the attributes of both, keeps the same underlying DOM node, and only updates the changed attributes. After handling the DOM node, React then recurses on the children.

- component elements of the same type: the instance stays the same, so that state is maintained across renders. React updates the props of the underlying component instance.

  - recursing on children => use keys to prevent re-creating children of random sequence. Keys should be stable, predictable, and unique.

# Virtual DOM

# Event Mechanism

## SyntheticEvent

A cross-browser wrapper around the browser’s native event. Use the ``nativeEvent`` attribute to get the underlying browser event.

## Event Pooling

The SyntheticEvent is pooled. This means that the SyntheticEvent object will be reused and all properties will be nullified after the event callback has been invoked.

Call ``event.persist()`` on the event to access the event properties in an asynchronous way, this will remove the synthetic event from the pool and allow references to the event to be retained by user code.

# Server-side rendering

# Web components
