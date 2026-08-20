# Senior Frontend Engineer — Technical Interview Master Notes
## Topic: `useContext()` Hook — Tricky Code & Output-Based Interview Masterclass (JavaScript Edition)

---

# 1. Definition

`useContext` is a core React Hook that enables functional components to subscribe to and consume values from a React Context Provider without explicit prop drilling. It reads the current context value from the nearest matching `<Context.Provider value={...}>` ancestor in the Fiber tree and automatically schedules a re-render for the subscribing component whenever that `value` reference changes (`Object.is`).

---

# 2. Pointwise Explanation (Senior-Level)

1. **Fiber Context Dependency Graph**: When `useContext` runs, it creates a `ContextDependency` node and appends it to the consuming Fiber's `dependencies` linked list, establishing a direct subscription.
2. **Bypasses `React.memo` Boundaries**: Context updates penetrate through intermediate memoized components (`React.memo`, `shouldComponentUpdate`, `PureComponent`) without requiring parent re-renders.
3. **Reference Equality Invalidation**: Value changes are determined strictly via `Object.is(prevValue, nextValue)`. Passing un-memoized object literals (`value={{ user, theme }}`) breaks render optimization.
4. **Nearest Ancestor Traversal**: Traverses upward along the Fiber tree hierarchy to locate the nearest matching Context Provider; if no matching Provider exists, it falls back to `createContext(defaultValue)`.
5. **No Granular Value Selectors**: React Context lacks built-in selector subscriptions (unlike Redux/Zustand). If a single property on a shared context object changes, **all consumers of that context re-render**.
6. **Provider Nesting & Overriding**: Context values can be scoped and overridden at different subtree levels by nesting multiple Providers for the same Context.
7. **Read-Only Subscription Contract**: `useContext` only reads state. Mutating state requires exposing updater callbacks or a `dispatch` function through the context payload.
8. **React 19 `use(Context)` Evolution**: In React 19, the `use(Context)` API can be called conditionally inside loops and `if` statements, unlike `useContext` which adheres strictly to Hook Rules.
9. **Propagation Mechanics (`propagateContextChange`)**: When a Provider's value changes, React scans all descendant Fibers during `beginWork`, matching `fiber.dependencies` to mark subscribed consumers as dirty (`lanes`).
10. **Decoupling Dependency**: Shields deep presentation leaves from structural refactoring in intermediate container components.

---

# 3. Why Do We Use It?

### Problem Solved
In deeply nested component hierarchies, sharing global or subtree-scoped state (such as authenticated user data, themes, localization, or active dashboard filters) via standard props requires **Prop Drilling**—threading props through intermediate components that do not need the data, polluting component contracts and creating rigid coupling.

### Without `useContext`
- Intermediate components become cluttered with boilerplate props (`user`, `setUser`, `theme`, `locale`).
- Refactoring layout containers breaks data pipelines across the tree.
- Adding a single global property requires editing dozens of intermediate files.

### When to Use
- **Subtree-wide, low-to-medium frequency state**: Active theme, authentication/session tokens, locale/i18n dictionaries, UI configuration flags.
- **Compound Components**: Accordions, Tabs, Select Dropdowns, Modal Dialogs sharing internal coordination state.
- **Dependency Injection**: Injecting API clients, analytics trackers, or logging interfaces for clean testing/mocking.

### When NOT to Use
- **High-frequency, streaming updates**: Live stock tickers, canvas mouse coordinates, WebSocket ticks (use dedicated state stores or ref subscriptions to avoid massive re-render storms).
- **Localized Parent-Child State**: When data is only needed 1–2 levels down (standard props or component composition/slots are cleaner).
- **As a replacement for Server Caches**: Fetching and caching API data should use TanStack Query or SWR, not giant global contexts.

---

# 4. Real-Time Production Scenarios

### Scenario 1: Multi-Tenant Enterprise Authentication & RBAC Engine
- **Requirement**: An enterprise SaaS application where 200+ views need immediate access to current user claims, active tenant ID, permissions, and a `hasPermission(role)` validator.
- **Problem**: Passing authentication state via props through 8 layers of dashboard layout, navigation drawers, and data tables created severe prop drilling and maintenance bottlenecks.
- **Solution**: Implemented an `AuthContext` with a custom `useAuth()` hook providing memoized `{ user, tenant, permissions, hasPermission }`.
- **Outcome**: UI subcomponents read access permissions directly at the point of consumption; layout components remain pure and decoupled.

### Scenario 2: Theme & Localization (i18n) Engine with Dynamic Overrides
- **Requirement**: A design system supporting dark/light modes and multi-language translations, with specific sub-sections (e.g., an embedded invoice preview) locked to a light theme.
- **Problem**: Global CSS variables alone did not allow programmatic React styling adjustments or localized date/currency formatters in dynamic canvas components.
- **Solution**: Built nested `ThemeContext.Provider` and `LocaleContext.Provider` layers. The root provided the global theme, while the invoice container nested an overriding Provider `<ThemeContext.Provider value="light">`.
- **Outcome**: The invoice sub-tree rendered seamlessly in light mode without affecting the outer dark theme, leveraging React's native hierarchical Provider resolution.

---

# 5. Visual Architecture Diagrams

#### Diagram 1: Context Propagation Through Memoization Boundaries

```
+-----------------------------------------------------------------------------------+
|                    Context Invalidation Through React.memo Boundaries             |
+-----------------------------------------------------------------------------------+

                          ┌───────────────────────────┐
                          │   App (ThemeProvider)     │  <--- State: theme = 'dark'
                          └─────────────┬─────────────┘
                                        │
                                        ▼
                          ┌───────────────────────────┐
                          │    MemoizedLayout (HOC)   │  <--- Wrapped in React.memo
                          │  (Props Unchanged -> Bail)│  <--- SKIPS RE-RENDER!
                          └─────────────┬─────────────┘
                                        │
                                        ▼
                          ┌───────────────────────────┐
                          │     MemoizedSidebar       │  <--- Wrapped in React.memo
                          │  (Props Unchanged -> Bail)│  <--- SKIPS RE-RENDER!
                          └─────────────┬─────────────┘
                                        │
                                        ▼
                          ┌───────────────────────────┐
                          │      ThemeToggleButton    │  <--- Reads useContext(ThemeContext)
                          │   (Direct Dependency)     │  <--- RE-RENDERS DIRECTLY!
                          └───────────────────────────┘
```

#### Diagram 2: Context Value Traversal & Fiber Dependency Graph

```
Fiber Tree Lookup:
Consumer Fiber -> dependencies.firstContext -> ContextDependency { context, next }
                                                     │
                                                     ▼
Traverse Ancestors: Find nearest matching Fiber tagged 'ContextProvider'
    ├── Found: Read Provider.pendingProps.value
    └── Not Found: Fallback to defaultValue in createContext(defaultValue)
```

---

# 6. Five Code Examples (JavaScript / JSX)

### Example 1 — Basic: Global Theme Consumption with Custom Hook
```jsx
import React, { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light');

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  const toggleTheme = () => setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within a ThemeProvider');
  return context;
};

export const ThemedButton = () => {
  const { theme, toggleTheme } = useTheme();
  return (
    <button 
      onClick={toggleTheme}
      style={{ background: theme === 'dark' ? '#333' : '#fff', color: theme === 'dark' ? '#fff' : '#000' }}
    >
      Current Theme: {theme}
    </button>
  );
};
```
- **Explanation**: Encapsulates theme state and toggle callback inside a Provider and guarded custom hook.
- **Output**: Clicking button toggles theme between `'light'` and `'dark'`.
- **Interview Point**: Exporting custom hooks ensures runtime provider existence checks and clean API boundaries.

### Example 2 — Practical: Compound Component Pattern (Tabs Engine)
```jsx
import React, { createContext, useContext, useState } from 'react';

const TabsContext = createContext(null);

export const Tabs = ({ defaultIndex = 0, children }) => {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);
  return (
    <TabsContext.Provider value={{ activeIndex, setActiveIndex }}>
      <div className="tabs-container">{children}</div>
    </TabsContext.Provider>
  );
};

export const Tab = ({ index, children }) => {
  const { activeIndex, setActiveIndex } = useContext(TabsContext);
  return (
    <button className={activeIndex === index ? 'active' : ''} onClick={() => setActiveIndex(index)}>
      {children}
    </button>
  );
};

export const TabPanel = ({ index, children }) => {
  const { activeIndex } = useContext(TabsContext);
  if (activeIndex !== index) return null;
  return <div className="tab-panel">{children}</div>;
};
```
- **Explanation**: Coordinates tab switching state implicitly across child components without prop drilling.
- **Output**: Clicking tabs mounts and displays matching tab panel content.
- **Interview Point**: Enables flexible declarative component layout rearrangement.

### Example 3 — Real-Time: Context Splitting for High-Performance State
```jsx
import React, { createContext, useContext, useReducer, memo } from 'react';

const StateContext = createContext(null);
const DispatchContext = createContext(null);

function counterReducer(state, action) {
  return action.type === 'INC' ? { count: state.count + 1 } : state;
}

export const CounterProvider = ({ children }) => {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  return (
    <DispatchContext.Provider value={dispatch}>
      <StateContext.Provider value={state}>{children}</StateContext.Provider>
    </DispatchContext.Provider>
  );
};

export const Display = () => {
  const { count } = useContext(StateContext);
  console.log('Display Rendered');
  return <h1>{count}</h1>;
};

export const IncBtn = memo(() => {
  const dispatch = useContext(DispatchContext);
  console.log('IncBtn Rendered (only once!)');
  return <button onClick={() => dispatch({ type: 'INC' })}>Increment</button>;
});
```
- **Explanation**: Separates volatile state from invariant dispatch context.
- **Output**: Clicking button increments count without re-rendering `IncBtn`.
- **Interview Point**: Standard pattern to eliminate unnecessary consumer re-renders.

### Example 4 — Advanced: Memoizing Context Value Payloads
```jsx
import React, { createContext, useContext, useState, useMemo, useCallback } from 'react';

const FilterContext = createContext(null);

export const FilterProvider = ({ children }) => {
  const [query, setQuery] = useState('');
  const [, setTick] = useState(0);

  const updateQuery = useCallback((text) => setQuery(text), []);

  const value = useMemo(() => ({ query, updateQuery }), [query, updateQuery]);

  return (
    <FilterContext.Provider value={value}>
      <button onClick={() => setTick(t => t + 1)}>Force Provider Re-render</button>
      {children}
    </FilterContext.Provider>
  );
};

export const SearchField = () => {
  const { query, updateQuery } = useContext(FilterContext);
  console.log('SearchField rendered');
  return <input value={query} onChange={(e) => updateQuery(e.target.value)} />;
};
```
- **Explanation**: Wrapping provider value in `useMemo` preserves object identity across unrelated provider parent re-renders.
- **Output**: Clicking "Force Provider Re-render" does not re-render `SearchField`.
- **Interview Point**: Prevents re-render cascades caused by inlined object literals.

### Example 5 — Interview-Level: Provider Scope Overriding & React 19 `use(Context)`
```jsx
import React, { createContext, useContext, use } from 'react';

const ScopeContext = createContext('root-default');

const ValueDisplay = () => {
  const val = typeof use === 'function' ? use(ScopeContext) : useContext(ScopeContext);
  return <div>Scope: {val}</div>;
};

export const MultiScope = () => (
  <div>
    <ValueDisplay />
    <ScopeContext.Provider value="tenant-A">
      <ValueDisplay />
      <ScopeContext.Provider value="tenant-B">
        <ValueDisplay />
      </ScopeContext.Provider>
    </ScopeContext.Provider>
  </div>
);
```
- **Explanation**: Shows hierarchical ancestor resolution and modern React 19 `use(Context)` conditional hook consumption.
- **Output**: Displays `Scope: root-default`, `Scope: tenant-A`, `Scope: tenant-B`.
- **Interview Point**: React resolves context from the nearest matching ancestor in the Fiber hierarchy.

---

# 7. How Does It Work Internally?

1. **Fiber Dependency Graph (`dependencies`)**: When calling `useContext(MyContext)`, React attaches a `ContextDependency` record (`{ context: MyContext, memoizedValue: value, next }`) to `currentlyRenderingFiber.dependencies.firstContext`.
2. **Provider Value Change Detection**: During `updateContextProvider`, React compares `Object.is(oldProps.value, newProps.value)`. If values differ, it invokes `propagateContextChange`.
3. **The `propagateContextChange` Algorithm**: React walks down the descendant Fiber subtree from the Provider, inspecting each Fiber's `dependencies`. When a matching Context dependency is found, it marks `fiber.lanes` and traverses upward marking `childLanes` on all ancestor Fibers.
4. **Bypassing Bailout**: Because the reconciler checks `childLanes` on every Fiber during `beginWork`, it will descend straight into the consumer Fiber to re-render it, completely bypassing intermediate `React.memo` bailout checkpoints.

---

# 8. Advantages vs. 9. Limitations

| Advantages | Disadvantages & Limitations |
| :--- | :--- |
| **Eliminates Prop Drilling** across deep trees | **The Selector Problem**: No property-level subscriptions |
| **Pierces `React.memo` boundaries** directly | **Re-render cascades** on inlined object literals |
| **Subtree-scoped provider overrides** | **Performance bottleneck** for high-frequency streaming |
| **Native Dependency Injection** for clean testing | **Context Hell / Provider Pyramids** at root |
| **Zero external bundle weight** (Built-in) | **Implicit coupling** makes unit testing isolated leaves harder |

---

# 10. Common Mistakes & Fixes

1. **Inlined Provider Value Objects**: `<Ctx.Provider value={{ user }}>` ➔ Always wrap context value objects in `useMemo`.
2. **Single Monolithic Context**: Storing auth, cart, theme, and alerts in one context ➔ Partition into discrete domain contexts (`AuthContext`, `CartContext`, `ThemeContext`).
3. **High-Frequency Streaming**: Placing mouse move coordinates in Context ➔ Use local state, refs, or external stores with selector subscriptions (Zustand).
4. **Missing Provider Defensive Guards**: Calling `useContext` without a provider returning `undefined` ➔ Add runtime guard inside custom hook: `if (!ctx) throw new Error(...)`.
5. **Overusing Context for Shallow Props**: Threading data 1 level down via Context ➔ Use Component Composition / Slots (`children` prop).

---

# 11. Best Practices

1. **Always Split State and Dispatch Contexts**: Isolate read-only state from invariant updater functions.
2. **Memoize Context Value Payloads**: Always use `useMemo` for provider `value` objects and `useCallback` for functions.
3. **Encapsulate in Custom Hooks**: Never export the raw Context object; export a custom hook with runtime safety guards.
4. **Co-locate Context with Feature Modules**: Keep context definitions close to their consuming features rather than dumping all at the root.
5. **Keep Context Domains Small & Granular**: Adhere to Single Responsibility; partition state into discrete domains.

---

# 12. Tricky Interview Questions (15 Code + Output Questions)

### Basic Level

#### Question 1: Un-Memoized Provider Object Reference Invalidation
```jsx
const UserContext = React.createContext();

const Profile = React.memo(() => {
  const { user } = React.useContext(UserContext);
  console.log('Profile rendered');
  return <h2>{user.name}</h2>;
});

export const App = () => {
  const [user] = React.useState({ name: 'Alex' });
  const [dummy, setDummy] = React.useState(0);

  return (
    <UserContext.Provider value={{ user }}>
      <button onClick={() => setDummy(d => d + 1)}>Dummy: {dummy}</button>
      <Profile />
    </UserContext.Provider>
  );
};
```
**Output upon clicking "Dummy":**
- Console logs: `"Profile rendered"`
- **Why:** `value={{ user }}` creates a brand-new object reference `{ user }` in memory on every `App` render. Even though `Profile` is wrapped in `React.memo` and `user` data didn't change, `Object.is(oldValue, newValue)` evaluates to `false`, forcing `Profile` to re-render.

---

#### Question 2: Default Value Fallback vs. Undefined Provider Value
```jsx
const AuthContext = React.createContext({ status: 'DEFAULT_GUEST' });

const UserBadge = () => {
  const auth = React.useContext(AuthContext);
  return <span>{auth ? auth.status : 'NO_AUTH'}</span>;
};

export const App = () => {
  return (
    <div>
      {/* Case 1: No Provider */}
      <UserBadge />

      {/* Case 2: Explicitly passing undefined */}
      <AuthContext.Provider value={undefined}>
        <UserBadge />
      </AuthContext.Provider>
    </div>
  );
};
```
**Output:**
- Case 1: `DEFAULT_GUEST`
- Case 2: `NO_AUTH`
- **Why:** In Case 1, because there is no matching Provider above it, React uses the default fallback passed to `createContext()`. In Case 2, a Provider *is* present and its value is explicitly `undefined`. `useContext` returns `undefined`, which does **not** fall back to the default value!

---

#### Question 3: Context Update Piercing Through `React.memo` Bailout
```jsx
const ThemeContext = React.createContext('light');

const DeepChild = () => {
  const theme = React.useContext(ThemeContext);
  console.log('DeepChild rendered:', theme);
  return <div>{theme}</div>;
};

const MiddleParent = React.memo(() => {
  console.log('MiddleParent rendered');
  return <DeepChild />;
});

export const App = () => {
  const [theme, setTheme] = React.useState('light');

  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={() => setTheme('dark')}>Toggle Theme</button>
      <MiddleParent />
    </ThemeContext.Provider>
  );
};
```
**Output upon clicking "Toggle Theme":**
- Console logs: `"DeepChild rendered: dark"` (Only DeepChild!)
- **Why:** `MiddleParent` has no props changed, so its `React.memo` check bails out of re-rendering `MiddleParent`. However, React's internal `propagateContextChange` traverses down and marks `DeepChild` dirty directly, updating it without re-rendering `MiddleParent`.

---

#### Question 4: Nearest Hierarchical Provider Resolution
```jsx
const NumberContext = React.createContext(0);

const Display = () => {
  const num = React.useContext(NumberContext);
  return <span>{num} </span>;
};

export const App = () => (
  <NumberContext.Provider value={10}>
    <Display />
    <NumberContext.Provider value={20}>
      <Display />
      <NumberContext.Provider value={30}>
        <Display />
      </NumberContext.Provider>
    </NumberContext.Provider>
  </NumberContext.Provider>
);
```
**Output:**
- UI displays: `10 20 30`
- **Why:** `useContext` resolves values by traversing upward from the consumer Fiber to find the *closest matching ancestor* Provider in the Fiber tree.

---

#### Question 5: Direct Mutation Failure
```jsx
const CountContext = React.createContext({ count: 0 });

const Child = () => {
  const ctx = React.useContext(CountContext);
  console.log('Child count:', ctx.count);
  return <div>{ctx.count}</div>;
};

export const App = () => {
  const stateRef = React.useRef({ count: 0 });

  const handleMutate = () => {
    stateRef.current.count += 1; // Direct mutation
  };

  return (
    <CountContext.Provider value={stateRef.current}>
      <button onClick={handleMutate}>Mutate</button>
      <Child />
    </CountContext.Provider>
  );
};
```
**Output upon clicking "Mutate":**
- UI stays `0`, console logs **nothing**.
- **Why:** Mutating object properties does not schedule a render on `App`, and passing the same object reference keeps `Object.is(prev, next) === true`, triggering zero context propagation.

---

### Intermediate Level

#### Question 6: The Context Selector Bottleneck
```jsx
const UserContext = React.createContext();

const UserName = React.memo(() => {
  const { name } = React.useContext(UserContext);
  console.log('UserName rendered');
  return <h3>{name}</h3>;
});

const UserAge = React.memo(() => {
  const { age } = React.useContext(UserContext);
  console.log('UserAge rendered');
  return <h3>{age}</h3>;
});

export const App = () => {
  const [user, setUser] = React.useState({ name: 'Alex', age: 25 });

  return (
    <UserContext.Provider value={user}>
      <button onClick={() => setUser(u => ({ ...u, age: u.age + 1 }))}>
        Increment Age
      </button>
      <UserName />
      <UserAge />
    </UserContext.Provider>
  );
};
```
**Output upon clicking "Increment Age":**
- Console logs:
  - `"UserName rendered"`
  - `"UserAge rendered"`
- **Why:** React Context does not support property-level selector subscriptions. Because `user` object identity changed, **all** consumers subscribing to `UserContext` re-render, even if the property they consume (`name`) didn't change.

---

#### Question 7: Context Provider Inside `ReactDOM.createPortal`
```jsx
import ReactDOM from 'react-dom';

const ColorContext = React.createContext('red');

const ModalContent = () => {
  const color = React.useContext(ColorContext);
  console.log('Modal color:', color);
  return <div style={{ color }}>Portal Modal</div>;
};

export const App = () => {
  return (
    <ColorContext.Provider value="blue">
      {ReactDOM.createPortal(
        <ModalContent />,
        document.body
      )}
    </ColorContext.Provider>
  );
};
```
**Output:**
- Console logs: `"Modal color: blue"`
- **Why:** Context resolution follows the **React Fiber tree hierarchy**, not the physical HTML DOM tree. Even though the DOM node lives in `document.body`, its Fiber parent is inside the `<ColorContext.Provider value="blue">`.

---

#### Question 8: Stale State Snapshot in Unmemoized Custom Hook Handlers
```jsx
const CounterContext = React.createContext();

const Child = () => {
  const { count, increment } = React.useContext(CounterContext);
  return <button onClick={increment}>Count: {count}</button>;
};

export const App = () => {
  const [count, setCount] = React.useState(0);

  // Intentional bug: Empty dependency array in parent handler
  const increment = React.useCallback(() => {
    setCount(count + 1); // Captures count = 0
  }, []);

  return (
    <CounterContext.Provider value={{ count, increment }}>
      <Child />
    </CounterContext.Provider>
  );
};
```
**Output after clicking button 3 times:**
- UI displays: `Count: 1`
- **Why:** `increment` captured `count = 0` in its closure when mounted. Every click evaluates `setCount(0 + 1) = 1`. To fix, use functional state updates: `setCount(prev => prev + 1)`.

---

#### Question 9: Context Provider Consuming Sibling Context
```jsx
const AuthContext = React.createContext({ user: 'Guest' });
const ProfileContext = React.createContext();

const ProfileProvider = ({ children }) => {
  const auth = React.useContext(AuthContext);
  const profile = { title: auth.user + ' Profile' };
  return (
    <ProfileContext.Provider value={profile}>
      {children}
    </ProfileContext.Provider>
  );
};

const ProfileDisplay = () => {
  const { title } = React.useContext(ProfileContext);
  return <h2>{title}</h2>;
};

export const App = () => {
  const [user, setUser] = React.useState('Alex');

  return (
    <AuthContext.Provider value={{ user }}>
      <ProfileProvider>
        <button onClick={() => setUser('Sam')}>Switch User</button>
        <ProfileDisplay />
      </ProfileProvider>
    </AuthContext.Provider>
  );
};
```
**Output after clicking "Switch User":**
- UI updates to: `Sam Profile`
- **Why:** When `App` updates `AuthContext`, `ProfileProvider` re-renders because it consumes `AuthContext`, calculating a new `profile` value and propagating it to `ProfileDisplay`.

---

#### Question 10: State and Dispatch Context Splitting Efficacy
```jsx
const StateCtx = React.createContext();
const DispatchCtx = React.createContext();

const Display = () => {
  const state = React.useContext(StateCtx);
  console.log('Display rendered');
  return <p>{state}</p>;
};

const ActionBtn = React.memo(() => {
  const dispatch = React.useContext(DispatchCtx);
  console.log('ActionBtn rendered (Only once!)');
  return <button onClick={() => dispatch('NEW_VAL')}>Click</button>;
});

export const App = () => {
  const [state, setState] = React.useState('INITIAL');

  return (
    <DispatchCtx.Provider value={setState}>
      <StateCtx.Provider value={state}>
        <Display />
        <ActionBtn />
      </StateCtx.Provider>
    </DispatchCtx.Provider>
  );
};
```
**Output upon clicking "Click":**
- Console logs:
  - `"Display rendered"`
  - (ActionBtn does **NOT** log!)
- **Why:** `setState` has a stable reference identity guaranteed by React. `ActionBtn` only consumes `DispatchCtx` and is wrapped in `React.memo`, so it completely bails out of re-rendering.

---

### Advanced / Tricky Level

#### Question 11: Provider Parent Re-render with Children Optimization
```jsx
const CountContext = React.createContext(0);

const Consumer = () => {
  const count = React.useContext(CountContext);
  console.log('Consumer rendered');
  return <div>Count: {count}</div>;
};

const Layout = ({ children }) => {
  const [tick, setTick] = React.useState(0);
  console.log('Layout rendered');

  return (
    <div>
      <button onClick={() => setTick(t => t + 1)}>Tick: {tick}</button>
      {children}
    </div>
  );
};

export const App = () => {
  return (
    <CountContext.Provider value={100}>
      <Layout>
        <Consumer />
      </Layout>
    </CountContext.Provider>
  );
};
```
**Output upon clicking "Tick":**
- Console logs: `"Layout rendered"`
- (Consumer does **NOT** log!)
- **Why:** `Consumer` was created in `App` and passed via `props.children` to `Layout`. When `Layout` re-renders, `props.children` maintains the identical JSX element reference created by `App`, so React skips re-rendering `Consumer`.

---

#### Question 12: Conditional Context Consumption with React 19 `use()`
```jsx
import { createContext, use, useState } from 'react';

const PremiumContext = createContext({ discount: '20%' });

const Checkout = ({ isPremium }) => {
  let discountText = '0%';

  // React 19 'use()' can be called conditionally!
  if (isPremium) {
    const { discount } = use(PremiumContext);
    discountText = discount;
  }

  return <div>Discount: {discountText}</div>;
};

export const App = () => {
  const [premium, setPremium] = useState(false);

  return (
    <PremiumContext.Provider value={{ discount: '50%' }}>
      <button onClick={() => setPremium(p => !p)}>Toggle Premium</button>
      <Checkout isPremium={premium} />
    </PremiumContext.Provider>
  );
};
```
**Output:**
- Initial: `Discount: 0%`
- After clicking button: `Discount: 50%`
- **Why:** Unlike `useContext`, React 19's `use(Context)` can be called inside conditionals (`if`), loops, and after early returns without violating React Hook rules.

---

#### Question 13: Eager Value Comparison Bailout in Providers
```jsx
const DataContext = React.createContext();

const Display = () => {
  const data = React.useContext(DataContext);
  console.log('Display rendered with:', data);
  return <div>{data}</div>;
};

export const App = () => {
  const [count, setCount] = React.useState(0);

  // Passing primitive value directly
  return (
    <DataContext.Provider value={42}>
      <button onClick={() => setCount(c => c + 1)}>Increment: {count}</button>
      <Display />
    </DataContext.Provider>
  );
};
```
**Output upon clicking "Increment":**
- Console output: **Nothing logged!**
- **Why:** During reconciliation, `updateContextProvider` checks `Object.is(42, 42) === true`. Because the value reference is strictly equal, React performs an eager bailout and does not invoke `propagateContextChange`.

---

#### Question 14: Context Read in Component Mounting Before Provider Mounts (Async Boundary)
```jsx
const AsyncContext = React.createContext('FALLBACK');

const Consumer = () => {
  const val = React.useContext(AsyncContext);
  console.log('Consumer saw:', val);
  return <div>{val}</div>;
};

export const App = () => {
  const [mounted, setMounted] = React.useState(false);

  React.useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) {
    return <Consumer />; // Mounted before Provider wraps it
  }

  return (
    <AsyncContext.Provider value="DYNAMIC_VALUE">
      <Consumer />
    </AsyncContext.Provider>
  );
};
```
**Output on Mount & Effect Resolution:**
- Initial Render: Console logs `"Consumer saw: FALLBACK"`
- Post-Effect Render: Console logs `"Consumer saw: DYNAMIC_VALUE"`
- **Why:** On initial mount, `Consumer` has no matching Provider ancestor and reads `'FALLBACK'`. When `mounted` becomes `true`, it is re-mounted inside `AsyncContext.Provider`, resolving `'DYNAMIC_VALUE'`.

---

#### Question 15: Overwriting Context Value with Same Inner Scope
```jsx
const ConfigContext = React.createContext({ theme: 'dark', lang: 'en' });

const Child = () => {
  const config = React.useContext(ConfigContext);
  console.log('Config:', config);
  return <div>{config.theme} - {config.lang}</div>;
};

export const App = () => {
  return (
    <ConfigContext.Provider value={{ theme: 'light', lang: 'en' }}>
      <ConfigContext.Provider value={{ theme: 'dark' }}>
        <Child />
      </ConfigContext.Provider>
    </ConfigContext.Provider>
  );
};
```
**Output:**
- Console logs: `Config: { theme: 'dark' }`
- UI displays: `dark - ` (lang is `undefined`!)
- **Why:** The inner Provider completely shadows the outer Provider. React does **not** merge nested Context values. If the inner Provider omits `lang`, it is `undefined`.

---

# 13. Output-Based Quick Reference Table

| Question # | Core Concept Tested | Primary Interview Trap |
| :--- | :--- | :--- |
| **Q1** | Object Reference Volatility | `value={{ user }}` creates new reference on every render |
| **Q2** | Default Value vs `undefined` | Passing `value={undefined}` does NOT use default fallback |
| **Q3** | Piercing `React.memo` | Context invalidates consumer without re-rendering memo parent |
| **Q4** | Nearest Ancestor Traversal | Context resolves from nearest Provider, not root |
| **Q5** | Direct Mutation Failure | `Object.is` check evaluates true; React skips render |
| **Q6** | The Selector Problem | Changing 1 property re-renders all consumers of that context |
| **Q7** | Portal Traversal | Context follows Fiber tree hierarchy, not DOM tree |
| **Q8** | Stale Closures | Missing deps in parent callback leads to stale state snapshots |
| **Q9** | Context Chaining | Provider reading another Context propagates updates cleanly |
| **Q10** | State/Dispatch Splitting | Isolating dispatch prevents buttons from re-rendering |
| **Q11** | Children Slot Optimization | `props.children` elements retain identity across parent renders |
| **Q12** | React 19 `use(Context)` | Can be called conditionally inside `if` statements |
| **Q13** | Eager Value Bailout | Unchanged primitive provider values trigger zero child work |
| **Q14** | Fallback Lifecycles | Unwrapped renders read fallback before provider mounts |
| **Q15** | Provider Shadowing | Nested providers shadow outer providers completely without merging |

---

# 14. Comparison With Alternatives

| Feature / Pattern | `useContext` | Props (Drilling) | Redux / RTK | Zustand | TanStack Query |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Primary Scope** | Global / Subtree State | Direct Parent-Child | Enterprise Global App | Lightweight Global App | Server Cache & Async Data |
| **Granular Selectors** | No (Full re-render) | Yes (Explicit props) | Yes (`useSelector`) | Yes (`useStore(selector)`) | Yes (Query selectors) |
| **Setup Complexity** | Low (Built-in) | Zero | High | Low | Low |
| **Bundle Footprint** | 0 KB (Built-in) | 0 KB | ~11 KB | ~1.5 KB | ~12 KB |
| **Update Frequency** | Low-to-Medium | Any | High | High | Server Ticks |
| **When to Use** | Themes, Auth, i18n, Compound UI | 1–2 levels data flow | Large multi-team web apps | Fast client-side state | All HTTP / REST / GraphQL APIs |

---

# 15. Senior-Level Explanation (30–45 Seconds)

> "`useContext` is React's native mechanism for broadcasting state across a component subtree without prop drilling. When a component consumes a context, React registers a direct Fiber dependency, allowing value changes to penetrate right through `React.memo` boundaries.
>
> In production, context is ideal for low-to-medium frequency updates like themes, auth sessions, and compound components. However, senior engineers must watch out for the Context Selector Problem: since React lacks property-level subscriptions, changing one property re-renders every consumer. We mitigate this by splitting state from dispatch, partitioning contexts by domain, and always memoizing provider values."

---

# 16. Deep-Dive Explanation (2–3 Minutes)

> "To explain `useContext` thoroughly, we must look at how React builds dependency graphs, manages propagation, and where its architectural boundaries lie.
>
> **Fiber Dependency Architecture**:
> When a component invokes `useContext(MyContext)`, React attaches a `ContextDependency` node to `currentlyRenderingFiber.dependencies`. This is not an event emitter; it's a direct pointer within Fiber's internal dependency list.
>
> **The Propagation Mechanism**:
> When a Provider's value changes, React executes `Object.is(oldValue, newValue)` during `updateContextProvider`. If they differ, React runs `propagateContextChange`, walking down the descendant Fiber tree to find matching context dependencies. It marks those consumer Fibers with `renderLanes` and flags all ancestor nodes with `childLanes`. This ensures the reconciler visits and updates the consumer even if intermediate components are wrapped in `React.memo`.
>
> **Production Best Practices & Caveats**:
> 1. **Value Memoization**: Always memoize provider value payloads with `useMemo`. If you pass an inline object literal `value={{ user, token }}`, every parent render creates a new object reference, causing a render cascade across all consumers.
> 2. **Context Splitting**: Because Context lacks built-in selectors, updating `user.age` will re-render components that only care about `user.name`. For high-scale apps, we split Contexts into atomic domains and separate `StateContext` from `DispatchContext`.
> 3. **React 19 Evolution**: In React 19, the new `use(Context)` API allows reading context conditionally inside `if` statements and loops, providing greater flexibility over traditional hook rules.
>
> In summary, `useContext` is a clean, native dependency-injection and state propagation engine, provided you respect reference equality and partition your state domains."

---

# 17. One-Line Interview Definition

> **"useContext is a native React Hook that subscribes functional components to the nearest matching Context Provider, automatically triggering re-renders whenever the provider's value changes reference."**

---

# 18. Interview Cheat Sheet

- **Definition**: Core Hook that consumes values from the nearest matching `<Context.Provider>`.
- **Why**: Eliminates prop drilling for subtree-wide data like auth, theme, and compound UI state.
- **How**: Adds `ContextDependency` to Fiber; Provider changes trigger `propagateContextChange` to mark consumers dirty.
- **Real-Time Use**: Authentication/RBAC, theme toggling, compound tabs/modals, i18n localization.
- **Key Advantage**: Pierces through `React.memo` boundaries directly to the consumer.
- **Key Limitation**: No built-in selector subscriptions; updating one property re-renders all consumers.
- **Common Mistake**: Passing inlined object literals `value={{ ... }}` without `useMemo`.
- **Most Important Point**: Split `StateContext` and `DispatchContext` to prevent action triggers from re-rendering on state updates.
- **Top 5 Tricky Questions**:
  1. *Does `React.memo` stop Context updates from reaching children?* (No, Context bypasses memo bailouts)
  2. *What is the return value if no Provider exists?* (The default value passed to `createContext`)
  3. *Why does Context cause re-render storms?* (Lack of granular property selectors)
  4. *What is the difference between `useContext` and React 19 `use(Context)`?* (`use(Context)` can be called conditionally)
  5. *Does Context resolve across `createPortal`?* (Yes, Context follows the React Fiber tree, not the DOM tree)
