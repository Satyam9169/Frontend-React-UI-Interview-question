# Redux Toolkit (RTK)

## What is Redux Toolkit?

Redux Toolkit (RTK) is the official, recommended way to write Redux
applications. It simplifies Redux development by reducing boilerplate
code and providing built-in best practices.

**Installation**

``` bash
npm install @reduxjs/toolkit react-redux
```

------------------------------------------------------------------------

# Why was Redux Toolkit introduced?

Traditional Redux required a lot of repetitive code:

-   Action Types
-   Action Creators
-   Reducers
-   Store Configuration
-   Middleware Setup

Even a simple counter application required multiple files.

**Traditional Redux Flow**

    Action Type
        ↓
    Action Creator
        ↓
    Reducer
        ↓
    Combine Reducers
        ↓
    Create Store
        ↓
    Provider
        ↓
    Dispatch Action

------------------------------------------------------------------------

# Problems Solved by Redux Toolkit

## 1. Reduces Boilerplate

### Traditional Redux

``` javascript
const INCREMENT = "INCREMENT";

const increment = () => ({
  type: INCREMENT
});

function reducer(state = 0, action) {
  switch(action.type){
    case INCREMENT:
      return state + 1;
    default:
      return state;
  }
}
```

### Redux Toolkit

``` javascript
const counterSlice = createSlice({
  name: "counter",
  initialState: 0,
  reducers: {
    increment(state) {
      state++;
    }
  }
});
```

------------------------------------------------------------------------

## 2. Simplified Store Configuration

### Traditional Redux

``` javascript
createStore(
  rootReducer,
  applyMiddleware(thunk)
);
```

### Redux Toolkit

``` javascript
const store = configureStore({
  reducer: {
    counter: counterReducer
  }
});
```

------------------------------------------------------------------------

## 3. Immutable Updates Using Immer

Instead of:

``` javascript
return {
  ...state,
  count: state.count + 1
}
```

RTK allows:

``` javascript
state.count++;
```

Immer safely creates a new immutable state behind the scenes.

------------------------------------------------------------------------

## 4. Built-in Async Support

``` javascript
export const fetchUsers = createAsyncThunk(
  "users/fetch",
  async () => {
    const response = await fetch("/users");
    return response.json();
  }
);
```

------------------------------------------------------------------------

# Why Use Redux Toolkit?

-   Global state management
-   Avoid prop drilling
-   Predictable application state
-   Redux DevTools support
-   Cleaner and maintainable code

------------------------------------------------------------------------

# Redux Toolkit Flow

    User Click
        ↓
    dispatch(action)
        ↓
    Slice Reducer
        ↓
    Store Updates
        ↓
    React Re-renders

------------------------------------------------------------------------

# Internal Working

1.  Create Store using `configureStore()`
2.  Create Slice using `createSlice()`
3.  Dispatch Action
4.  Reducer updates state
5.  Immer creates immutable state
6.  Store updates
7.  Components using `useSelector()` re-render

------------------------------------------------------------------------

# Important Functions

## configureStore()

``` javascript
const store = configureStore({
  reducer: {
    counter: counterReducer
  }
});
```

## createSlice()

``` javascript
const counterSlice = createSlice({
  name: "counter",
  initialState: {
    value: 0
  },
  reducers: {
    increment(state) {
      state.value++;
    }
  }
});
```

## useSelector()

``` javascript
const count = useSelector(
  state => state.counter.value
);
```

## useDispatch()

``` javascript
const dispatch = useDispatch();

dispatch(increment());
```

## createAsyncThunk()

``` javascript
export const fetchUsers = createAsyncThunk(
  "users/fetch",
  async () => {
    const response = await fetch("/users");
    return response.json();
  }
);
```

------------------------------------------------------------------------

# Advantages

-   Less boilerplate
-   Official Redux recommendation
-   Easy store configuration
-   Automatic action generation
-   Built-in Immer
-   Built-in Redux DevTools
-   Built-in Thunk middleware
-   Organized code using slices
-   Excellent TypeScript support

------------------------------------------------------------------------

# Disadvantages

-   Overkill for very small apps
-   Learning curve for Redux concepts
-   Large global state can become difficult to manage
-   Slight bundle size increase

------------------------------------------------------------------------

# Real-world Example

An e-commerce application:

-   Navbar
-   Product List
-   Wishlist
-   Cart
-   Checkout
-   Profile

When a user clicks **Add to Cart**:

1.  `dispatch(addToCart(product))`
2.  Cart slice updates the store.
3.  Navbar cart count updates automatically.
4.  Cart page displays the new item.
5.  Checkout uses the same cart state.

------------------------------------------------------------------------

# Interview Answer (2--4 Years Experience)

Redux Toolkit is the official and recommended way to use Redux. It
simplifies state management by reducing boilerplate code through
utilities like `configureStore`, `createSlice`, and `createAsyncThunk`.
It helps manage global state, avoids prop drilling, provides predictable
state updates, and uses Immer internally for immutable updates.
Components dispatch actions, reducers update the store, and subscribed
components automatically re-render. It improves maintainability,
debugging, and scalability of React applications.
