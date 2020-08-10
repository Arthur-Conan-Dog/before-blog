# Hooks

## Hooks FAQ

### [Do Hooks replace render props and higher-order components?](https://reactjs.org/docs/hooks-faq.html#do-hooks-replace-render-props-and-higher-order-components)

> Often, render props and higher-order components render only a single child. We think Hooks are a simpler way to serve this use case. There is still a place for both patterns (for example, a virtual scroller component might have a renderItem prop, or a visual container component might have its own DOM structure). But in most cases, Hooks will be sufficient and can help reduce nesting in your tree.

> Idiomatic code using Hooks doesn’t need the deep component tree nesting that is prevalent in codebases that use higher-order components, render props, and context. With smaller component trees, React has less work to do.

### How do I implement getDerivedStateFromProps?

正常地写对比 prevProps & currentProps 的逻辑就可以，prevProps 可能通过 useState 创建 state object 来记录。

### [How do I implement shouldComponentUpdate?](https://reactjs.org/docs/hooks-faq.html#how-do-i-implement-shouldcomponentupdate)

You can wrap a function component with React.memo to shallowly compare its props. `React.Memo` only compares props.

You can also add a second argument to specify a custom comparison function that takes the old and new props. If it returns true, the update is skipped.

### [Is there something like forceUpdate?](https://reactjs.org/docs/hooks-faq.html#is-there-something-like-forceupdate)

You can use an incrementing counter to force a re-render even if the state has not changed:

```jsx
const [ignored, forceUpdate] = useReducer((x) => x + 1, 0);

function handleClick() {
  forceUpdate();
}
```

### [Can I make a ref to a function component?](https://reactjs.org/docs/hooks-reference.html#useimperativehandle)

```jsx
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

A parent component that renders <FancyInput ref={fancyInputRef} /> would be able to call fancyInputRef.current.focus().

### [How can I measure a DOM node?](https://reactjs.org/docs/hooks-faq.html#how-can-i-measure-a-dom-node)

Use a callback ref:

```jsx
function MeasureExample() {
  const [rect, ref] = useClientRect();
  return (
    <>
      <h1 ref={ref}>Hello, world</h1>
      {rect !== null && (
        <h2>The above header is {Math.round(rect.height)}px tall</h2>
      )}
    </>
  );
}

function useClientRect() {
  const [rect, setRect] = useState(null);
  const ref = useCallback((node) => {
    if (node !== null) {
      setRect(node.getBoundingClientRect());
    }
  }, []);
  return [rect, ref];
}
```

### [Is it safe to omit functions from the list of dependencies?](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)

Of course no. The problem is what is the correct way to put functions into dependencies array.

- Declare your function inside `useEffect`, and the dependencies become data that function uses.

If for some reason you can’t move a function inside an effect, there are a few more options:

- You can try moving that function outside of your component. In that case, the function is guaranteed to not reference any props or state, and also doesn’t need to be in the list of dependencies.

- If the function you’re calling is a pure computation and is safe to call while rendering, you may call it outside of the effect instead, and make the effect depend on the returned value.

- As a last resort, you can add a function to effect dependencies but wrap its definition into the useCallback Hook. This ensures it doesn’t change on every render unless its own dependencies also change.

### [How to create expensive objects lazily?](https://reactjs.org/docs/hooks-faq.html#how-to-create-expensive-objects-lazily)

- when creating the initial state is expensive: pass a function to `useState`

- avoid re-creating the useRef() initial value: write your own function that creates and sets it lazily

### [How to avoid passing callbacks down?](https://reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down)

1. create a context

2. create a reducer and get its dispatch using `useReducer`

3. set dispatch as value prop of Context.Provider

4. get dispatch in child component using `useContext`

Note that you can still choose whether to pass the application state down as props (more explicit) or as context (more convenient for very deep updates).

### How to create custom hooks with dependency array?

Only need arguments, implement caching logic inside using hooks with dependency array such as useEffect or useCallback.
