# Advanced React Patterns

## Higher-Order Components(HOC)

A higher-order component is a function that takes a component and returns a new component. The alternative plan for mixins.

A HOC composes the original component by wrapping it in a container component.

Usage example: different components that need to subscribe to the same data source, wrap logic of subscription in the container component.

Purpose: 关注点分离 => 逻辑复用。

Conventions:

* Separate props of different usages

  ```jsx
  render() {
    const { extraProp, ...passThroughProps } = this.props;
    const injectedProp = someStateOrInstanceMethod;

    return (
      <WrappedComponent
        injectedProp={injectedProp}
        {...passThroughProps}
      />
    );
  }
  ```

* Maximizing Composability: Single-argument HOCs have the signature Component => Component. Functions whose output type is the same as its input type are really easy to compose together.

* Configure a display name for easier debugging

Caveats:

* Don’t Use HOCs Inside the render Method: or it will be re-created every render, and the internal states will be lost.

* Static Methods Must Be Copied Over:

  * mannually set on HOC or use tools such as hoist-non-react-statics to copy automatically

  * export the static method separately from the component itself

* Refs Aren’t Passed Through

  * forwardRef

## Render Props

The term “render prop” refers to a coding pattern for sharing render logic between React components using a prop whose value is a function.

Usage example:

```jsx
<Parent render={(props) => <Child {...props}/>}>

// or...

<Parent>
  {(props) => <Child {...props}/>}
</Parent>
```

Purpose: 关注点分离 => 逻辑复用。条件渲染，组件对外暴露自定义插槽接口。后者 hooks 能做到，但使用是否自然可能要看具体场景。

Conventions:

* You can implement most higher-order components (HOC) using a regular component with a render prop.

Caveats:

* Using a render prop can negate the advantage that comes from using React.PureComponent if you create the function inside a render method. To get around this problem, you can sometimes define the prop as an instance method. => - [ ] What about using children props?

## HOC vs Render Props vs Hooks

### [Comparison: HOC vs Render Props vs Hooks](https://medium.com/simply/comparison-hocs-vs-render-props-vs-hooks-55f9ffcd5dc6)

1. Define these patterns

2. Compare them from different perspectives: readability, reusability, customization and usage, debugging, testing, performance

3. Scoring levels: poor - fair - good - very good - excellent

4. Total score = developer experience

5. Conclusion & When to use

很好地给出了对比的维度，以及代码示例。大部分观点是在理的，我表示认同。

不太认同的点：

1. 关于 Performance，具体问题具体分析，并不能代表普遍情况下的性能，不过他也从渲染次数上说明了 hooks 的优势。

### [Avoiding HOC; Favoring render props](https://gist.github.com/heygrady/f9bf3b6dd93fe3d87ba87430fd3c20d5)

Reasons:

Breaking the rules of HOC:

* Not providing any derived props

* Not providing any callback props

Confusing and restrictive:

* Creates a new "wrapped" component, complicating the React tree

* Only supports a single child, must be a component

* Requires additional compatibility code to work like a "normal" component

Compatibility code:

* Copy over static methods

* forward refs

* wrapping the display name to make debug easier

### [Discussions on Reddit](https://www.reddit.com/r/reactjs/comments/azo7tm/confused_about_the_state_harhar_of_hoc_v_render/)

> Lastly, my rule of thumb here is, whenever you need to share logic between components, not some UI (that's what components are for), use hooks.

### From Maintainer

> One of the design goals of Hooks is to avoid the deeply nested functional style that is prevalent with higher-order components and render props.

> Passing values between Hooks is at the heart of our proposal. Render props pattern was the closest you could get to it without Hooks, but you couldn’t get full benefits without something like Component Component which has a lot of syntactic noise due to a “false hierarchy”. Hooks flatten that hierarchy to passing values — and function calls is the simplest way to do that.

https://overreacted.io/why-do-hooks-rely-on-call-order/#flaw-8-too-much-ceremony

### Tips

#### [Support both Hooks and Render Props with a simple trick](https://americanexpress.io/hydra/)

```jsx
import { useState } from 'react';
import renderProps from 'render-props';

const useCounter = initialCount => {
  const [count, setCount] = useState(initialCount);
  const deltaCount = delta => setCount(count => count + delta);

  return {
    count,
    incCount: () => deltaCount(1),
    decCount: () => deltaCount(-1),
  };
};

const Counter = ({ initialCount, children, render = children }) =>
  renderProps(render, useCounter(initialCount));

Counter.defaultProps = {
  initialCount: 0,
};

export default Counter;
export { useCounter };
```
