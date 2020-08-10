# Redux

### Why handling async calls is complex with Redux?

The philosophy of Redux, is to keep state immutable. It means state can only be replaced with next state, and thus cannot be mutated directly. The reducer function should be a pure function.

```javascript
console.log(store.getState()); // { isLoading: false }

store.dispatch({ type: 'loading' }); // dispatch(action)

console.log(store.getState()); // { isLoading: true }
```

We don't want to write plain action objects again and again, so we need to create functions that return different action types when being executed, those functions are called `actionCreators` in Redux world.

```javascript
export const startFetch = () => ({
  type: 'loading',
});
```

Further more, extract `loading` string into a constant.

```javascript
export const START_LOADING = 'start loading';
// ...
import * as types from './types';
```

And reducer is just the handler of dispatched action of a certain type.

However, async calls will bring side effects and break this purity. Redux provide middlewares as a solution to this issue. In that case we could not only return plain javascript objects in action creators, but also promises or functions.

### Why I can get return value from calling dispatch async actions, aren't they handled internally?

Actually the return value is just the action created by action creator function, it could be a plain JavaScript object, or a promise, or undefined. I think return action after executing dispatch is to fullfill the propose of chaining middlewares.

### `redux-thunk` & `redux-promise` etc, what are the differences?

They are only different approaches to enable Redux to accept things other than plain object actions.

### How does Redux apply middlewares?

See: https://github.com/Arthur-Conan-Dog/redux-middleware
