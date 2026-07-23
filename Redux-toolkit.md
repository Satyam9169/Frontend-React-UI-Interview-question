# Redux Toolkit

The **Redux Toolkit (RTK)** package is the official, recommended way to
write Redux logic. It was created to solve three common Redux problems:

1.  Configuring a Redux store is too complicated.
2.  Too many packages are required to use Redux effectively.
3.  Redux requires too much boilerplate code.

------------------------------------------------------------------------

# Redux Toolkit APIs

## 1. configureStore()

### Key Points

1.  Creates the Redux Store.
2.  Automatically adds Redux Thunk middleware.
3.  Enables Redux DevTools and combines reducers.

### Code Example

``` javascript
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "./counterSlice";

const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

export default store;
```

------------------------------------------------------------------------

## 2. createReducer()

### Key Points

1.  Creates reducers without using switch statements.
2.  Uses Immer, allowing mutable-looking state updates.
3.  Reduces boilerplate and makes reducers cleaner.

``` javascript
import { createReducer } from "@reduxjs/toolkit";

const initialState = { count: 0 };

const counterReducer = createReducer(initialState, (builder) => {
  builder.addCase("increment", (state) => {
    state.count++;
  });

  builder.addCase("decrement", (state) => {
    state.count--;
  });
});
```

------------------------------------------------------------------------

## 3. createAction()

### Key Points

1.  Automatically creates action creator functions.
2.  Reduces manual action type code.
3.  Prevents action type spelling mistakes.

``` javascript
import { createAction } from "@reduxjs/toolkit";

const increment = createAction("counter/increment");

dispatch(increment());
```

------------------------------------------------------------------------

## 4. createSlice()

### Key Points

1.  Creates reducers automatically.
2.  Creates action creators automatically.
3.  Keeps state, reducers, and actions together.

``` javascript
import { createSlice } from "@reduxjs/toolkit";

const counterSlice = createSlice({
  name: "counter",
  initialState: { count: 0 },
  reducers: {
    increment(state) {
      state.count++;
    },
    decrement(state) {
      state.count--;
    },
  },
});

export const { increment, decrement } = counterSlice.actions;
export default counterSlice.reducer;
```

------------------------------------------------------------------------

## 5. combineSlices()

### Key Points

1.  Combines multiple slice reducers.
2.  Simplifies store configuration.
3.  Supports lazy loading of slices.

``` javascript
import { combineSlices } from "@reduxjs/toolkit";
import userSlice from "./userSlice";
import cartSlice from "./cartSlice";

const rootReducer = combineSlices(
  userSlice,
  cartSlice
);

export default rootReducer;
```

------------------------------------------------------------------------

## 6. createAsyncThunk()

### Key Points

1.  Simplifies API calls and async operations.
2.  Automatically creates pending, fulfilled, and rejected actions.
3.  Reduces async Redux boilerplate.

``` javascript
import { createAsyncThunk } from "@reduxjs/toolkit";

export const fetchUsers = createAsyncThunk(
  "users/fetchUsers",
  async () => {
    const response = await fetch("https://jsonplaceholder.typicode.com/users");
    return response.json();
  }
);
```

------------------------------------------------------------------------

## 7. createEntityAdapter()

### Key Points

1.  Stores data in a normalized format.
2.  Provides built-in CRUD helper methods.
3.  Improves performance when handling large collections.

``` javascript
import { createEntityAdapter } from "@reduxjs/toolkit";

const usersAdapter = createEntityAdapter();
const initialState = usersAdapter.getInitialState();
```

------------------------------------------------------------------------

## 8. createSelector()

### Key Points

1.  Creates memoized selectors.
2.  Recalculates only when state changes.
3.  Improves performance by avoiding unnecessary recalculations.

``` javascript
import { createSelector } from "@reduxjs/toolkit";

const selectUsers = (state) => state.users;

const activeUsers = createSelector(
  [selectUsers],
  (users) => users.filter((user) => user.active)
);
```

------------------------------------------------------------------------

# Redux Toolkit Architecture

``` text
React Component
      │
      │ useDispatch()
      ▼
Dispatch Action
      │
      ▼
createSlice / createAsyncThunk
      │
 ┌────┴──────────────┐
 │                   │
 ▼                   ▼
Sync Action      Async Action
                     │
                     ▼
 Pending / Fulfilled / Rejected
        │
        ▼
     Reducer
        │
        ▼
 Redux Store (configureStore)
        │
        ▼
 Updated State
        │
        ▼
 useSelector()
        │
        ▼
 React Re-renders UI
```

------------------------------------------------------------------------

# Real-Time Example

## Without Redux Toolkit

``` text
Product Component
      │
      ▼
Call API
      │
Store in local state
      │
Pass props to many components
      │
State becomes difficult to manage
```

## With Redux Toolkit

``` text
Product Component
      │
      ▼
dispatch(fetchProducts())
      │
      ▼
createAsyncThunk()
      │
      ▼
API Call
      │
      ▼
Redux Store
      │
      ▼
All Components use useSelector()
```

**Benefit:** No prop drilling.

------------------------------------------------------------------------

# Problems Solved

1.  **Too much boilerplate** → `createSlice()`
2.  **Complex async logic** → `createAsyncThunk()`
3.  **Prop drilling / state sharing** → Global Redux Store
4.  **Complex immutable updates** → Immer

------------------------------------------------------------------------

# Advantages

-   Less boilerplate
-   Easy API handling
-   Cleaner code
-   Uses Immer
-   Redux DevTools support
-   Scalable for large apps
-   Better performance with `createSelector()`
-   Standardized architecture

------------------------------------------------------------------------

# Disadvantages

-   Learning curve
-   Overkill for small apps
-   Global store can become large
-   Async debugging can be harder
-   Slight bundle size increase

------------------------------------------------------------------------

# Use Redux Toolkit When

-   E-commerce applications
-   Banking applications
-   Admin dashboards
-   CRM systems
-   Social media apps
-   Inventory management
-   Applications with shared global state

------------------------------------------------------------------------

# Avoid Redux Toolkit When

-   Portfolio websites
-   Landing pages
-   Small blogs
-   Simple calculators
-   Small forms using only local state

------------------------------------------------------------------------

# 2-Minute Interview Answer

Redux Toolkit is the official and recommended way to write Redux
applications. It reduces boilerplate, automatically configures the Redux
store, generates reducers and actions with `createSlice`, handles
asynchronous API calls using `createAsyncThunk`, and enables immutable
state updates using Immer. It solves problems such as prop drilling,
complex state management, and repetitive Redux setup. It is best suited
for medium to large applications where multiple components need to share
global state efficiently.
