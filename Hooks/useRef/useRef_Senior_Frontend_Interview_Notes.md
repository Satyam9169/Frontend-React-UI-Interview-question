# useRef() Hook in React — Senior Frontend Interview Notes

## 1. Definition

`useRef()` is a built-in React Hook that returns a **mutable reference object** whose `.current` property is initialized to the passed argument (`initialValue`). The returned object persists across renders for the entire lifetime of the component.

Crucially, **mutating the `.current` property does not trigger a component re-render**, and `useRef()` does not create a snapshot per render like `useState()`.

```jsx
const refContainer = useRef(initialValue);
// Returns: { current: initialValue }
```

---

# 2. Pointwise Explanation — Exactly 10 Points

1. **Persistent Mutable Container:** `useRef()` creates a plain JavaScript object `{ current: initialValue }` that React preserves across all render passes without resetting.

2. **No Re-renders on Mutation:** Unlike `useState()` or `useReducer()`, modifying `refContainer.current = newValue` does **not** notify React or schedule a re-render.

3. **DOM Element Access:** When passed to a JSX element via the `ref` prop (`<input ref={inputRef} />`), React automatically assigns the underlying DOM node to `.current` upon mounting and resets it to `null` on unmount.

4. **Synchronous Mutation & Immediate Value:** Updating `ref.current` is completely synchronous and immediately readable anywhere in the execution context, unlike `setState` which schedules updates for future renders.

5. **Instance Variable Equivalent:** For functional components, `useRef()` acts as the direct replacement for class component instance variables (`this.timerId`, `this.isMounted`, `this.previousState`).

6. **Escape Hatch from Pure Rendering:** Reading or writing `.current` during the JSX render phase violates pure rendering principles; ref mutations should strictly happen inside event handlers or `useEffect()` / `useLayoutEffect()`.

7. **`forwardRef` & Imperative Handles:** Custom components do not receive `ref` as a standard prop. `forwardRef` allows parent components to attach a ref to an inner child DOM node, and `useImperativeHandle` lets children customize the exposed ref methods.

8. **Callback Refs Alternative:** For dynamic lists or measuring DOM nodes that mount/unmount conditionally, a callback ref `ref={(node) => ...}` is preferred over `useRef()` because `useRef()` does not notify when the node is attached or detached.

9. **Previous State Tracking:** Because ref values are mutable and independent of render snapshots, storing a snapshot inside `useEffect()` allows tracking the previous render's props or state.

10. **`useRef()` vs Top-Level Variable:** Module-level variables persist globally across all component instances (causing cross-component data leakage), whereas `useRef()` is strictly isolated to the specific component instance.

---

# 3. Why Do We Use `useRef()`?

## Why does it exist?

React components are JavaScript functions that re-execute on every render. Any standard local variable declared inside the function body is re-created from scratch on each render:

```jsx
function Timer() {
  let timerId = null; // ❌ Reset to null on EVERY render!

  function start() {
    timerId = setInterval(() => console.log("Tick"), 1000);
  }

  function stop() {
    clearInterval(timerId); // Fails because timerId was reset if component re-rendered!
  }
}
```

`useState()` persists values across renders, but updating state always schedules a render. When you need to store data that changes over time **without affecting the UI**, `useState()` causes wasteful re-renders.

`useRef()` solves both problems:
1. Persists data between renders (unlike local variables).
2. Modifies data silently without triggering a render (unlike `useState`).

## What problem would exist without it?

Without `useRef()`:
- Managing imperative third-party DOM libraries (D3, Chart.js, Google Maps) would require unsafe global variables or DOM query selectors (`document.getElementById`).
- Storing timer identifiers (`setInterval`, `setTimeout`) or AbortControllers would trigger unnecessary re-renders.
- Tracking previous props or states would require awkward state duplication.
- Accessing actual DOM nodes for focus, scrolling, animations, or measurements would be error-prone.

## When should we use it?

```text
- Managing focus, text selection, media playback, or canvas contexts
- Storing timer IDs (setInterval, setTimeout)
- Storing mutable instance values (isMounted flags, AbortControllers, WebSocket references)
- Storing previous props or state for comparison
- Integrating with non-React imperative libraries
- Caching values where changes do not affect JSX output
```

## When should we NOT use it?

1. **Rendering data in JSX:** If a value needs to be displayed on the screen, use `useState()` or compute it directly. Mutating a ref will not update the DOM.
2. **Synchronous Render Mutations:** Never do `ref.current = count + 1` inside the main function body during render.

---

# 4. Real-Time Production Scenarios

## Scenario 1 — Video Player Custom Controls

### Requirement
A custom video player dashboard requires Play, Pause, Fast Forward (+10s), and Mute buttons interacting directly with HTML5 `<video>` imperative APIs without triggering React re-renders.

### Solution

```jsx
import { useRef, useState } from "react";

function CustomVideoPlayer({ src }) {
  const videoRef = useRef(null);
  const [isPlaying, setIsPlaying] = useState(false);

  const togglePlay = () => {
    if (!videoRef.current) return;

    if (videoRef.current.paused) {
      videoRef.current.play();
      setIsPlaying(true);
    } else {
      videoRef.current.pause();
      setIsPlaying(false);
    }
  };

  const skipForward = () => {
    if (videoRef.current) {
      videoRef.current.currentTime += 10;
    }
  };

  return (
    <div className="player-container">
      <video ref={videoRef} src={src} width="600" />
      <div className="controls">
        <button onClick={togglePlay}>{isPlaying ? "Pause" : "Play"}</button>
        <button onClick={skipForward}>+10s</button>
      </div>
    </div>
  );
}
```

---

## Scenario 2 — Preventing Initial-Mount Side Effect Execution

### Requirement
An analytics tracking hook or auto-save feature needs to run when state updates, but **must skip running on the initial mount**.

### Solution

```jsx
import { useEffect, useRef } from "react";

function useUpdateEffect(callback, dependencies) {
  const isFirstRender = useRef(true);

  useEffect(() => {
    if (isFirstRender.current) {
      isFirstRender.current = false;
      return;
    }

    return callback();
  }, dependencies);
}
```

---

# 5. Six Production Code Examples

## Example 1 — Imperative DOM Focus Management

```jsx
import { useRef, useEffect } from "react";

function SearchModal({ isOpen }) {
  const searchInputRef = useRef(null);

  useEffect(() => {
    if (isOpen && searchInputRef.current) {
      searchInputRef.current.focus();
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div className="modal">
      <input ref={searchInputRef} type="text" placeholder="Type to search..." />
    </div>
  );
}
```

---

## Example 2 — Timer / Stopwatch with Accurate Cleanup

```jsx
import { useState, useRef, useEffect } from "react";

function Stopwatch() {
  const [seconds, setSeconds] = useState(0);
  const timerRef = useRef(null);

  const startTimer = () => {
    if (timerRef.current !== null) return; // Prevent duplicate timers

    timerRef.current = setInterval(() => {
      setSeconds((s) => s + 1);
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(timerRef.current);
    timerRef.current = null;
  };

  const resetTimer = () => {
    stopTimer();
    setSeconds(0);
  };

  // Ensure teardown on unmount
  useEffect(() => {
    return () => clearInterval(timerRef.current);
  }, []);

  return (
    <div>
      <h3>Elapsed: {seconds}s</h3>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
      <button onClick={resetTimer}>Reset</button>
    </div>
  );
}
```

---

## Example 3 — Custom `usePrevious` Hook (Tracking Previous Props/State)

```jsx
import { useEffect, useRef } from "react";

function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value; // Updated after render commits
  }, [value]);

  return ref.current; // Returns previous value during render phase
}

// Consuming Component:
function PriceTicker({ price }) {
  const prevPrice = usePrevious(price);
  const trend = price > prevPrice ? "▲" : "▼";

  return (
    <p>
      Current: ${price} (Prev: ${prevPrice}) {prevPrice !== undefined && trend}
    </p>
  );
}
```

---

## Example 4 — `forwardRef` and `useImperativeHandle`

```jsx
import { forwardRef, useImperativeHandle, useRef } from "react";

const CustomTextInput = forwardRef((props, ref) => {
  const nativeInputRef = useRef(null);

  // Expose only specific methods to parent instead of raw DOM element
  useImperativeHandle(ref, () => ({
    focusAndSelect: () => {
      nativeInputRef.current?.focus();
      nativeInputRef.current?.select();
    },
    clearInput: () => {
      if (nativeInputRef.current) nativeInputRef.current.value = "";
    },
  }));

  return <input ref={nativeInputRef} type="text" {...props} />;
});

// Parent Usage:
function Form() {
  const inputRef = useRef(null);

  return (
    <div>
      <CustomTextInput ref={inputRef} placeholder="Enter name" />
      <button onClick={() => inputRef.current?.focusAndSelect()}>Select</button>
      <button onClick={() => inputRef.current?.clearInput()}>Clear</button>
    </div>
  );
}
```

---

## Example 5 — Callback Refs for Dynamic Elements & Measurements

```jsx
import { useState, useCallback } from "react";

function MeasureElement() {
  const [height, setHeight] = useState(0);

  // Callback ref is called with the DOM node on mount and null on unmount
  const measuredRef = useCallback((node) => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <div>
      <h2 ref={measuredRef}>Measured Heading</h2>
      <p>The above heading is {Math.round(height)}px tall.</p>
    </div>
  );
}
```

---

## Example 6 — Avoiding Stale Closures in Asynchronous Callbacks

```jsx
import { useState, useEffect, useRef } from "react";

function StaleClosureFix() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);

  // Keep ref synchronized with latest count on every render
  useEffect(() => {
    countRef.current = count;
  }, [count]);

  const handleDelayedLog = () => {
    setTimeout(() => {
      // count -> stale snapshot
      // countRef.current -> always latest current value
      console.log("Latest Count via Ref:", countRef.current);
    }, 3000);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
      <button onClick={handleDelayedLog}>Log after 3s</button>
    </div>
  );
}
```

---

# 6. How Does `useRef()` Work Internally?

In React Fiber's internal architecture, `useRef()` is one of the simplest hooks:

```text
mountRef(initialValue)
         ↓
Create plain JS object: { current: initialValue }
         ↓
Attach object to Fiber hook's memoizedState
         ↓
Return hook.memoizedState
```

During subsequent re-renders:

```text
updateRef()
         ↓
Read Fiber hook's memoizedState
         ↓
Return the EXACT SAME object reference { current: ... }
```

### Key Internal Insights:
1. **Plain Object Identity:** React does not wrap `useRef` in reactive proxies, getters, or setters. It is literally `{ current: value }`.
2. **Persistent Fiber Reference:** Because the object reference in `hook.memoizedState` never changes, JavaScript identity equality (`Object.is(prevRef, nextRef) === true`) holds across every render pass.
3. **DOM Attachment Timing:** React attaches DOM elements to refs during the **Commit Phase** (mutation pass) before running `useLayoutEffect` and `useEffect`. React detaches DOM refs (`ref.current = null`) during the unmount mutation pass.

---

# 7. Advantages

1. **Zero Re-renders:** Modifying `.current` updates data instantly without triggering expensive render cycles.
2. **Persistent Component Memory:** Preserves values across the entire lifecycle of a component instance.
3. **Direct DOM Interaction:** Clean, declarative way to access native browser DOM nodes and APIs.
4. **Synchronous Reads & Writes:** Reading or updating `.current` gives immediate real-time values without waiting for React re-render passes.
5. **Prevents Stale Closures:** Solves closure stale-data problems in async callbacks (`setTimeout`, `Promise`, event listeners).
6. **Encapsulated Scope:** Unlike global variables, refs are strictly isolated per component instance.
7. **Instance State Storage:** Perfect for non-visual properties (timer IDs, abort signals, WebSocket sockets).
8. **Interoperability:** Bridges declarative React with imperative third-party UI libraries (D3, Mapbox, TinyMCE).
9. **Controlled Ref Exposure:** Integrates cleanly with `useImperativeHandle` for strict parent-child API boundaries.
10. **Lightweight & High Performance:** Minimal memory overhead in React's Fiber data structure.

---

# 8. Disadvantages / Limitations

1. **No UI Synchronization:** Changing `.current` will never update the visual JSX on the screen.
2. **Silent Failure if Render Depends on It:** Developers mistakenly use refs for UI state and wonder why the screen doesn't update.
3. **Not Pure / Render-Phase Mutation Risks:** Writing or reading `ref.current` during the render phase introduces non-deterministic rendering bugs.
4. **No Change Notification:** React provides no listener or subscription when `.current` changes (unlike `useState`).
5. **Manual Lifecycle Management:** Timers or sockets stored in refs must be manually cleared/closed on unmount.
6. **Passing Refs Requires `forwardRef`:** Prior to React 19, passing refs to custom components requires wrapping them in `forwardRef`.
7. **DOM Null Trap:** `ref.current` is `null` on the initial render until React commits the DOM tree.
8. **Not Suitable for Conditional DOM Tracking:** Static `useRef()` does not trigger callbacks when a DOM node mounts or unmounts conditionally (requires callback refs).
9. **Break Encapsulation:** Overusing DOM refs to manipulate styles or values imperatively violates declarative component design.
10. **State Duplication Risk:** Maintaining both a state variable and a ref for the same data can cause synchronization drift if not updated carefully.

---

# 9. Common Mistakes

## Mistake 1 — Reading or Writing `.current` During Rendering
### ❌ Wrong
```jsx
function Counter() {
  const countRef = useRef(0);
  countRef.current++; // Modifying during render phase breaks React purity!
  return <h1>{countRef.current}</h1>;
}
```
### ✅ Correct
```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return <h1>{count}</h1>;
}
```

---

## Mistake 2 — Expecting UI to Update on Ref Mutation
### ❌ Wrong
```jsx
function App() {
  const count = useRef(0);
  return (
    <button onClick={() => { count.current += 1; }}>
      Clicks: {count.current} {/* Will NOT update on click! */}
    </button>
  );
}
```
### ✅ Correct
```jsx
function App() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(c => c + 1)}>Clicks: {count}</button>
  );
}
```

---

## Mistake 3 — Accessing DOM Node Before Mount
### ❌ Wrong
```jsx
function Input() {
  const inputRef = useRef(null);
  inputRef.current.focus(); // 💥 TypeError: inputRef.current is null on initial render!
  return <input ref={inputRef} />;
}
```
### ✅ Correct
```jsx
function Input() {
  const inputRef = useRef(null);
  useEffect(() => {
    inputRef.current?.focus(); // Safe inside useEffect after DOM committed
  }, []);
  return <input ref={inputRef} />;
}
```

---

## Mistake 4 — Using Module-Level Global Variable Instead of `useRef`
### ❌ Wrong
```jsx
let globalTimer = null; // Shared across ALL instances of TimerComponent!

function TimerComponent() {
  const start = () => { globalTimer = setInterval(...); };
}
```
### ✅ Correct
```jsx
function TimerComponent() {
  const timerRef = useRef(null); // Isolated per component instance
  const start = () => { timerRef.current = setInterval(...); };
}
```

---

## Mistake 5 — Storing Expensive Initial Values Without Lazy Init
### ❌ Wrong
```jsx
// expensiveComputation() runs on EVERY single render pass!
const dataRef = useRef(expensiveComputation());
```
### ✅ Correct
```jsx
const dataRef = useRef(null);
if (dataRef.current === null) {
  dataRef.current = expensiveComputation(); // Runs only once
}
```

---

# 10. Best Practices

1. **Keep Renders Pure:** Never mutate or read `ref.current` during the JSX render phase. Restrict mutations to `useEffect`, `useLayoutEffect`, or event handlers.
2. **Use Refs for Non-Visual State:** Store timer IDs, WebSocket connections, AbortControllers, and cached calculations that don't directly alter the JSX output.
3. **Always Check for `null`:** Guard all DOM ref operations with optional chaining (`ref.current?.focus()`).
4. **Clean Up Mutable Resources:** Always clear intervals, timeouts, and listeners stored in refs when the component unmounts.
5. **Prefer Callback Refs for Dynamic DOM:** When you need to measure or manipulate a DOM element that renders conditionally, use a callback ref instead of `useRef`.
6. **Encapsulate Component Refs:** Use `forwardRef` with `useImperativeHandle` to expose safe, minimal imperative APIs to parents rather than full DOM access.

---

# 11. Tricky Interview Questions

## Basic — Question 1
**Question:** What is the structure of the object returned by `useRef(initialValue)`?  
**Answer:** It returns a plain JavaScript object: `{ current: initialValue }`.

---

## Basic — Question 2
**Question:** What happens to the UI when you update `ref.current = 100`?  
**Answer:** Nothing happens to the UI. React does not track mutations to `ref.current`, so no re-render is scheduled or performed.

---

## Intermediate — Question 3
**Question:** What is the difference between `useRef()` and `React.createRef()`?  
**Answer:**
- `useRef()`: Returns the **same persistent object instance** across all render passes in a functional component.
- `createRef()`: Creates a **brand new ref object on every single render**. In class components, `createRef()` was placed in the constructor (run once), but inside functional component bodies, `createRef()` would reset on every render.

---

## Intermediate — Question 4
**Question:** When during the React rendering lifecycle does React attach DOM nodes to `ref.current`?  
**Answer:** React sets `ref.current` during the **Commit Phase** (specifically when the DOM elements are mutated/created) before firing `useLayoutEffect` and `useEffect`.

---

## Advanced — Question 5
**Question:** How can you track the previous value of a prop or state variable using `useRef()`?  
**Answer:** Because `useEffect` runs **after** the render phase commits, you can return `ref.current` during the render phase and update `ref.current = value` inside `useEffect`. The render sees the old value before the effect updates it for the next pass.

---

# 12. Output-Based Interview Questions

## Output Question 1
```jsx
function App() {
  const count = useRef(0);
  const [renderCount, setRenderCount] = useState(0);

  const handleClick = () => {
    count.current += 1;
    console.log("Ref count:", count.current);
  };

  return (
    <div>
      <button onClick={handleClick}>Increment Ref</button>
      <button onClick={() => setRenderCount((c) => c + 1)}>Re-render</button>
      <p>Rendered: {count.current}</p>
    </div>
  );
}
```
**User Action:** Clicks "Increment Ref" 3 times, then clicks "Re-render" once.

**Output:**
```text
Console:
Ref count: 1
Ref count: 2
Ref count: 3

UI (Initial): Rendered: 0
UI (After 3 clicks on Ref): Rendered: 0 (No UI change)
UI (After clicking Re-render): Rendered: 3
```

---

## Output Question 2
```jsx
function Timer() {
  const [count, setCount] = useState(0);
  const ref = useRef(0);

  useEffect(() => {
    ref.current = count;
  });

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Click</button>
      <p>State: {count} | Ref: {ref.current}</p>
    </div>
  );
}
```
**Initial Render Output:** `State: 0 | Ref: 0`  
**After 1 Click:** `State: 1 | Ref: 0` (During render, `ref.current` is 0; effect updates it to 1 *after* render).  
**After 2nd Click:** `State: 2 | Ref: 1`.

---

# 13. Scenario-Based Interview Questions

## Scenario 1 — Dynamic List Focus
**Question:** You have a dynamic list of 50 todo items. Clicking "Edit" on any item must immediately focus that item's specific input field. How do you implement refs?  
**Answer:** Instead of creating 50 separate `useRef` calls, maintain a `useRef(new Map())` or an object dictionary:
```jsx
const itemRefs = useRef(new Map());

const focusItem = (id) => {
  const node = itemRefs.current.get(id);
  node?.focus();
};

// In JSX:
<input ref={(el) => (el ? itemRefs.current.set(item.id, el) : itemRefs.current.delete(item.id))} />
```

---

## Scenario 2 — Imperative Parent Trigger
**Question:** A parent component needs to trigger a `.resetForm()` function inside a child component without managing form fields in parent state.  
**Answer:** Use `forwardRef` in the child combined with `useImperativeHandle`:
```jsx
const ChildForm = forwardRef((props, ref) => {
  useImperativeHandle(ref, () => ({
    resetForm: () => { /* internal reset logic */ }
  }));
  return <form>...</form>;
});
```

---

# 14. Comparison With Alternatives

| Concept | Persists Across Renders? | Triggers Re-render on Update? | Mutable? | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **`useRef`** | **Yes** | **No** | **Yes** (`ref.current = x`) | DOM access, timers, non-visual persistent state |
| **`useState`** | **Yes** | **Yes** | **No** (Immutable setters) | UI data, state that drives JSX rendering |
| **Local Variable (`let x`)** | **No** (Resets every render) | **No** | **Yes** | Temporary calculations within a single render pass |
| **Module Variable (`let x` outside)** | **Yes** (Lives globally) | **No** | **Yes** | Shared app constants (⚠️ leaks across component instances) |
| **`createRef`** | **No** in functional components | **No** | **Yes** | Legacy class components constructor |

---

# 15. Senior-Level Explanation — 30–45 Seconds

> "`useRef` is a React Hook that returns a persistent, mutable container object with a `.current` property that survives component re-renders. Unlike `useState`, mutating `.current` is completely synchronous and never schedules or triggers a re-render. In senior frontend architectures, I use `useRef` for two primary purposes: accessing and managing imperative DOM elements (like focus, canvas, or media players), and storing instance variables like timer IDs, AbortControllers, and previous props that must persist without affecting visual rendering."

---

# 16. Deep-Dive Explanation — 2–3 Minutes

> "`useRef` fundamentally serves as an escape hatch in React functional components. When invoked, React creates a plain JavaScript object `{ current: initialValue }` and stores it on the Fiber node's `memoizedState`. Across subsequent render passes, React guarantees that it returns the exact same object reference.
>
> There are two foundational use cases for `useRef`:
> 1. **DOM Node Reference:** By passing the ref to a JSX element's `ref` prop, React imperatively binds the underlying DOM node during the commit phase before layout effects and passive effects fire. This enables focus management, animations, and measurements without breaking declarative React flow.
> 2. **Instance Variables & Lifecycle Data:** In functional components, local variables reset on every render, while `useState` causes re-renders on mutation. `useRef` bridges this gap by persisting non-visual data—such as interval IDs, WebSocket instances, `isMounted` boolean flags, or previous state snapshots—without incurring rendering overhead.
>
> As a senior engineer, I follow strict design guidelines:
> - **Pure Rendering:** Never read or mutate `ref.current` during the JSX render execution phase, as this violates React's concurrency assumptions.
> - **Exposing Minimal Imperative APIs:** When exposing child ref methods to parent components, I combine `forwardRef` with `useImperativeHandle` to prevent leaky abstractions.
> - **Callback Refs:** For dynamically rendered or conditional elements, I favor callback refs over `useRef` to handle DOM node attachments and detachments reactively."

---

# 17. One-Line Interview Definition

> **`useRef()` is a React Hook that provides a persistent, mutable reference object (`{ current: value }`) across renders without triggering component re-renders on mutation.**

---

# 18. Interview Cheat Sheet

- **Core Return:** `{ current: initialValue }`
- **Re-render on Mutation:** **Never** (mutations are silent and synchronous).
- **DOM Access:** `<input ref={inputRef} />` $ightarrow$ Attached in Commit Phase.
- **Access Guard:** Always check `ref.current !== null` before calling native DOM APIs.
- **Component Forwarding:** Wrap component with `forwardRef` to accept `ref` prop.
- **Top Anti-Pattern:** Never use `useRef` for values that must update JSX visuals on the screen.

---

# ⭐ Senior-Level Points Interviewers Look For

```text
useRef
  │
  ├── Persistent mutable container across renders
  ├── Zero re-renders on mutation
  ├── Synchronous read/write access
  ├── DOM element access during Commit phase
  ├── Instance variables (timer IDs, abort signals)
  ├── Stale closure mitigation in async callbacks
  ├── forwardRef & useImperativeHandle encapsulation
  ├── Callback refs for dynamic/conditional DOM nodes
  └── Pure render rule: no ref mutations in render body
```

---

# Final Interview Formula

When an interviewer asks:

> **"What is useRef and when do you use it?"**

Use this structure:

```text
Definition & Return Value ({ current: value })
    ↓
Key Differentiator (No re-renders + synchronous persistence)
    ↓
Primary Use Case 1 (DOM manipulation: focus, media, canvas)
    ↓
Primary Use Case 2 (Instance state: timers, abort signals, previous props)
    ↓
Render Purity Rule (Never mutate ref.current during render body)
    ↓
Advanced Patterns (forwardRef, useImperativeHandle, Callback refs)
    ↓
Comparison Summary (useRef vs useState vs local let variable)
```
