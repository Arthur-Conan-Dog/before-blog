# Fundamentals

## JSX

Syntax extension to JavaScript. JSX prevents injection attacks.

## Pure functions

不会更改输入；相同输入，相同输出。

## State

Class components should always call the base constructor with props. => [Why?](https://overreacted.io/why-do-we-write-super-props/)

State updates may be Asynchronous, and are merged.

## Forms

Controlled Components

HTML 中表单元素通常管理着它们自己的状态，并根据用户输入来更新。而在 React 中，可被更改的状态通常由 State 来管理。我们可以把两者结合起来，使 React 的 state 成为“唯一数据源”。

Uncontrolled Components

由 HTML 元素自己管理状态，React 只读其状态。

一个例外 => `<input type="file">` 由于 value 只读，其必须是 uncontrolled component。 => Q: Formik 中如何处理？
