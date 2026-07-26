ReactJS Hooks
--------------------
1) useState()
=> useState() is a React Hook that allows functional components to store and update state.

Syntax
const [state, setState] = useState(initialValue);

1) state → Current state value.
2) setState → Function used to update the state.
3) initialValue → Initial value of the state.


Example 1: Counter

import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

export default Counter;


Output:

Initial value: 0
Clicking the button updates the count.


Example 2: String State

import { useState } from "react";

function Greeting() {
  const [name, setName] = useState("John");

  return (
    <div>
      <h2>Hello, {name}</h2>
      <button onClick={() => setName("Alice")}>
        Change Name
      </button>
    </div>
  );
}

Example 3: Updated User Age

import React, { useState } from "react";

function UserProfile() {
  const [user, setUser] = useState({
    name: "John",
    age: 25,
    city: "New York",
  });

  const updateAge = () => {
    setUser({
      ...user,      // Copy all existing properties
      age: 26,      // Update only the age
    });
  };

// This avoids issues if multiple state updates are queued before React re-renders the component.
  const updateAge = () => {
     setUser((prevUser) => ({
     ...prevUser,
      age: prevUser.age + 1,
     }));
  }	



  return (
    <div>
      <h2>User Profile</h2>

      <p>Name: {user.name}</p>
      <p>Age: {user.age}</p>
      <p>City: {user.city}</p>

      <button onClick={updateAge}>
        Update Age
      </button>
    </div>
  );
}

export default UserProfile;

1) useState() can only be used inside functional components or custom hooks.
2) Updating state with setState causes the component to re-render.
3) State updates are asynchronous, so don't expect the state value to change immediately after calling setState.
4) useState stores component state.



2) useEffect() 

useEffect() is a React Hook used to perform side effects in functional components.

What are Side Effects?

Side effects are operations that happen outside of rendering the UI, such as:

✅ API calls
✅ Fetching data
✅ Timers (setTimeout, setInterval)
✅ Event listeners
✅ DOM manipulation
✅ Updating document title
✅ Cleanup (remove listeners, clear timers


Syntax:
--------
useEffect(() => {
  // Side effect code

  return () => {
    // Cleanup code (optional)
  };
}, [dependencies]);


Example 1: Runs on Every Render

import { useState, useEffect } from "react";

function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("Component Rendered");
  });

  return (
    <div>
      <h2>{count}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

export default App;

Output
Initial Render
Component Rendered

Click Button
Component Rendered

Click Again
Component Rendered

Example 2: Runs Only Once (Component Mount)
import { useEffect } from "react";

function App() {

  useEffect(() => {
    console.log("API Called");
  }, []);

  return <h1>Hello React</h1>;
}

export default App;

Output:
Component Mounted

API Called

Click Button (if any)
Nothing Happens

Reason: An empty dependency array ([]) means the effect runs only once after the first render.


Example 3: Runs When State Changes


import { useState, useEffect } from "react";

function App() {

  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("Count Changed:", count);
  }, [count]);

  return (
    <div>
      <h2>{count}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

export default App;


Output

Initial
Count Changed: 0

Click
Count Changed: 1

Click
Count Changed: 2

Reason: The effect runs initially and every time count changes.

Example 4: Multiple Dependencies

import { useState, useEffect } from "react";

function App() {

  const [count, setCount] = useState(0);
  const [name, setName] = useState("John");

  useEffect(() => {
    console.log("Count or Name Changed");
  }, [count, name]);

  return (
    <div>
      <h2>{count}</h2>
      <h2>{name}</h2>

      <button onClick={() => setCount(count + 1)}>
        Count
      </button>

      <button onClick={() => setName("Alice")}>
        Change Name
      </button>
    </div>
  );
}

export default App;

The effect runs whenever either count or name changes.

Example 5: Fetch API Data

import { useState, useEffect } from "react";

function App() {

  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/users")
      .then((response) => response.json())
      .then((data) => setUsers(data));
  }, []);

  return (
    <div>
      <h2>Users</h2>

      {users.map((user) => (
        <p key={user.id}>{user.name}</p>
      ))}
    </div>
  );
}

export default App;

Flow:
Component Mounts
      ↓
useEffect Executes
      ↓
API Call
      ↓
Data Received
      ↓
setUsers(data)
      ↓
Component Re-renders


Example 6: Cleanup Function (Event Listener)

import { useEffect } from "react";

function App() {

  useEffect(() => {

    const handleResize = () => {
      console.log(window.innerWidth);
    };

    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };

  }, []);

  return <h2>Resize Window</h2>;
}

export default App;

Why Cleanup?

Without cleanup:

Mount
↓

Listener Added

↓

Unmount

↓

Listener Still Exists ❌

With cleanup:

Mount

↓

Listener Added

↓

Unmount

↓

Listener Removed ✅

Example 7: Timer Cleanup
import { useEffect } from "react";

function App() {

  useEffect(() => {

    const timer = setInterval(() => {
      console.log("Running...");
    }, 1000);

    return () => {
      clearInterval(timer);
    };

  }, []);

  return <h2>Timer Example</h2>;
}

export default App;



Execution Order
Component Renders
        ↓
Browser Updates UI
        ↓
useEffect Runs

Dependency Array Summary
| Dependency      | Runs When                                           |
| --------------- | --------------------------------------------------- |
| No dependency   | After every render                                  |
| `[]`            | Only once after the initial render                  |
| `[count]`       | Initial render + whenever `count` changes           |
| `[count, name]` | Initial render + whenever `count` or `name` changes |

Interview Points
useEffect is used to perform side effects after rendering.
It runs after React updates the DOM.
An empty dependency array ([]) runs the effect only once after the initial render.
Dependencies control when the effect re-runs.
The cleanup function runs before the next effect (if dependencies changed) and when the component unmounts.
Common uses include API calls, timers, event listeners, subscriptions, and updating the document title.
In React 18+ Strict Mode (development only), effects may run twice on initial mount to help detect bugs. This does not happen in production.

3) useRef()
useRef() is a React Hook that returns a mutable object with a .current property. The value stored in .current persists across renders and changing it does not trigger a re-render.

Syntax:
const ref = useRef(initialValue);
            or
const inputRef = useRef(null);



Interview Points (Must Know)
useRef returns an object with a .current property.
2. The ref object persists across all renders.
3. Updating ref.current does not trigger a re-render.
4. It is commonly used to access DOM elements directly.
5. It is useful for storing mutable values like timers, sockets, and previous state.
6. The same ref object is reused on every render.
7. Use useState when the UI should update; use useRef when you need to keep data without re-rendering.
8. useRef is frequently used with useEffect for tasks like focusing an input after the component mounts.

Easy Way to Remember
useState → Data changes → UI updates (re-render)
useRef → Data changes → No UI update (no re-render), or direct access to DOM elements

Real-Time Examples
| Scenario                             | Hook       |
| ------------------------------------ | ---------- |
| Form input value displayed on screen | `useState` |
| Focus an input                       | `useRef`   |
| Scroll to a section                  | `useRef`   |
| Store previous count                 | `useRef`   |
| Store timer ID                       | `useRef`   |
| Store WebSocket instance             | `useRef`   |
| Play/Pause video                     | `useRef`   |
| Chart.js instance                    | `useRef`   |


useState vs useRef
| Feature               | useState | useRef |
| --------------------- | -------- | ------ |
| Stores data           | ✅        | ✅      |
| Causes re-render      | ✅ Yes    | ❌ No   |
| Access DOM            | ❌        | ✅      |
| Stores previous value | ❌        | ✅      |
| Stores timer ID       | ❌        | ✅      |
| Mutable object        | ❌        | ✅      |
| Used for UI updates   | ✅        | ❌      |


Example 1: Access an Input Field (Most Common)
import React, { useRef } from "react";

function App() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus();
  };

  return (
    <div>
      <input
        ref={inputRef}
        type="text"
        placeholder="Enter your name"
      />

      <button onClick={focusInput}>
        Focus Input
      </button>
    </div>
  );
}

export default App;
How it works

Initially

inputRef.current = null

After rendering

inputRef.current
      │
      ▼
<input />

Click Button

inputRef.current.focus();

Output

Cursor automatically moves inside the input.


Example 2: Store Previous Count
import React, { useState, useEffect, useRef } from "react";

function App() {
  const [count, setCount] = useState(0);

  const previousCount = useRef(0);

  useEffect(() => {
    previousCount.current = count;
  }, [count]);

  return (
    <div>
      <h2>Current Count : {count}</h2>
      <h2>Previous Count : {previousCount.current}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

export default App;

Initially

Current Count : 0
Previous Count : 0

Click 1

Current Count : 1
Previous Count : 0

Click 2

Current Count : 2
Previous Count : 1

Click 3

Current Count : 3
Previous Count : 2

Example 3: Count Renders Without Re-rendering
import React, { useState, useRef } from "react";

function App() {
  const [count, setCount] = useState(0);

  const renderCount = useRef(1);

  renderCount.current++;

  return (
    <div>
      <h2>Count : {count}</h2>

      <h2>Render Count : {renderCount.current}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

export default App;

Every render increases renderCount.current, but changing .current itself does not trigger a render.


Example 4: Store Timer ID
import React, { useRef } from "react";

function App() {
  const timerRef = useRef(null);

  const startTimer = () => {
    timerRef.current = setInterval(() => {
      console.log("Running...");
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(timerRef.current);
  };

  return (
    <div>
      <button onClick={startTimer}>
        Start
      </button>

      <button onClick={stopTimer}>
        Stop
      </button>
    </div>
  );
}

export default App;
The timer ID is stored in timerRef.current without causing re-renders.



Example 5: Change Input Value Directly
import React, { useRef } from "react";

function App() {
  const inputRef = useRef();

  const changeValue = () => {
    inputRef.current.value = "Hello React";
  };

  return (
    <div>
      <input ref={inputRef} />

      <button onClick={changeValue}>
        Change Value
      </button>
    </div>
  );
}

export default App;
Output:
Before Button

Input: __________

After Button

Input: Hello React

4) Custom Hooks in React
A Custom Hook is a JavaScript function whose name starts with use. It allows you to extract and reuse stateful logic across multiple components.

Rule: A custom hook must start with use, for example:
useFetch
useAuth
useWindowSize
useCounter

Syntax
function useCustomHook() {
  // React Hooks (useState, useEffect, useRef, etc.)

  return value;
}

Why do we use Custom Hooks? (7 Points)
1. Reuse Logic
Instead of writing the same code in multiple components, write it once in a custom hook.
2. Reduces Code Duplication.
3. Makes Components Cleaner
4. Easy to Maintain
5. Improves Readability
6. Follows the DRY Principle
DRY = Don't Repeat Yourself
7. Easy to Test => Testing becomes simpler because the business logic is separated from the UI.

Benefits of Custom Hooks
✅ Reusable logic
✅ Cleaner components
✅ Less duplicate code
✅ Easier maintenance
✅ Better readability
✅ Easier testing
✅ Better project structure
✅ Promotes modular code
✅ Improves scalability

Example 1: useCounter
// useCounter.js

import { useState } from "react";

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => {
    setCount((prev) => prev + 1);
  };

  const decrement = () => {
    setCount((prev) => prev - 1);
  };

  const reset = () => {
    setCount(initialValue);
  };

  return {
    count,
    increment,
    decrement,
    reset,
  };
}

export default useCounter;

Component
import React from "react";
import useCounter from "./useCounter";

function App() {
  const {
    count,
    increment,
    decrement,
    reset,
  } = useCounter(10);

  return (
    <div>
      <h2>Count: {count}</h2>

      <button onClick={increment}>
        +
      </button>

      <button onClick={decrement}>
        -
      </button>

      <button onClick={reset}>
        Reset
      </button>
    </div>
  );
}

export default App;

Interview Points
A custom hook is a JavaScript function that starts with use.
It allows you to reuse stateful logic across multiple components.
A custom hook can use other React hooks like useState, useEffect, useRef, and useReducer.
It does not render UI; it only encapsulates logic and returns values or functions.
Custom hooks help keep components clean, reusable, and easier to maintain.
They must follow the Rules of Hooks (call hooks only at the top level and only from React components or other custom hooks).
Common real-world custom hooks include useFetch, useAuth, useDebounce, useLocalStorage, and useWindowSize.


5) React.Memo()
React.memo() is a Higher-Order Component (HOC) that prevents unnecessary re-renders of a functional component.

It memoizes (remembers) the previous rendered result and only re-renders the component if its props change.

Syntax:
const MemoizedComponent = React.memo(Component);
or
export default React.memo(Component);

Why do we use React.memo()? (7 Points)
1. Prevents Unnecessary Re-renders ⭐

If a parent component re-renders but the child component's props haven't changed, React.memo() skips re-rendering the child.

2. Improves Performance ⭐

Large or expensive components don't need to render again when nothing has changed.

3. Memoizes the Component

React stores the previous rendered output and reuses it if the props are the same.

4. Works Only with Props

React.memo() compares previous props and new props.

Same props → No re-render
Different props → Re-render
5. Useful for Large Applications

Useful when:

Dashboard
Product List
Chat Application
Data Tables
Forms
6. Reduces Rendering Cost

Skipping unnecessary rendering improves application performance.

7. Supports Custom Comparison

You can provide your own comparison function if the default shallow comparison isn't enough.


Example 1: Without React.memo()
Child Component
import React from "react";

function Child() {
  console.log("Child Rendered");

  return <h2>Child Component</h2>;
}

export default Child;
Parent Component
import React, { useState } from "react";
import Child from "./Child";

function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h2>Count : {count}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>

      <Child />
    </div>
  );
}

export default App;
Console Output
Initial Render

Child Rendered

Click Increment

Child Rendered

Click Again

Child Rendered

❌ Even though the child doesn't use count, it still re-renders because the parent re-rendered.

Example 2: With React.memo()
Child Component
import React from "react";

function Child() {
  console.log("Child Rendered");

  return <h2>Child Component</h2>;
}

export default React.memo(Child);
Parent Component
import React, { useState } from "react";
import Child from "./Child";

function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h2>Count : {count}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>

      <Child />
    </div>
  );
}

export default App;
Console Output
Initial Render

Child Rendered

Click Increment

(No Child Render)

Click Again

(No Child Render)

✅ Since the child receives no changing props, React skips its re-render.

How React.memo() Works
Parent Re-renders
        │
        ▼
Compare Previous Props
        │
        ▼
Are Props Equal?
      /      \
    Yes       No
    │         │
Skip Render   Re-render Child

Benefits
✅ Improves performance
✅ Prevents unnecessary re-renders
✅ Reduces rendering cost
✅ Useful for large applications
✅ Easy to implement
✅ Uses shallow prop comparison by default
✅ Can use custom comparison logic

Real-Time Use Cases
| Component        | Why use `React.memo()`?                          |
| ---------------- | ------------------------------------------------ |
| Product Card     | Avoid re-rendering unchanged cards               |
| User Profile     | Skip updates if user data is unchanged           |
| Sidebar          | Doesn't need to re-render with every page update |
| Chat Message     | Existing messages remain unchanged               |
| Data Table Row   | Only changed rows should re-render               |
| Dashboard Widget | Expensive widgets can be skipped                 |
| Navigation Bar   | Prevent unnecessary renders                      |

6) useMemo()
useMemo() is a React Hook that memoizes (caches) the result of an expensive calculation. It recalculates the value only when its dependencies change, helping improve performance.


Syntax
const memoizedValue = useMemo(() => {
  // Expensive calculation
  return result;
}, [dependencies]);

Why do we use useMemo()? (7 Points)

1. Prevents Expensive Calculations ⭐

If a calculation takes time, useMemo() stores the result so it isn't recalculated on every render.

2. Improves Performance ⭐

Avoids unnecessary computations, making the application faster.

3. Memoizes Values

It remembers the previously calculated value until a dependency changes.

4. Recalculates Only When Dependencies Change
const total = useMemo(() => calculateTotal(items), [items]);

calculateTotal() runs only when items changes.

5. Prevents Unnecessary Work
Without useMemo, every parent render would recalculate the value, even if the input didn't change.

6. Helps with Stable Object/Array References

Useful when passing objects or arrays to a React.memo() component.

const user = useMemo(() => ({
  name: "John",
}), []);
7. Useful for Large Applications

Commonly used in:

Large tables
Filtering data
Sorting
Searching
Dashboard calculations
Charts



Example 1: Expensive Calculation
Without useMemo()
import React, { useState } from "react";

function App() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  const square = (() => {
    console.log("Calculating...");
    return count * count;
  })();

  return (
    <>
      <h2>Count: {count}</h2>
      <h2>Square: {square}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>

      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
    </>
  );
}

export default App;
Console
Initial

Calculating...

Type in Input

Calculating...

Type Again

Calculating...

❌ Even typing in the input recalculates square.

Example 2: With useMemo()
import React, { useMemo, useState } from "react";

function App() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  const square = useMemo(() => {
    console.log("Calculating...");
    return count * count;
  }, [count]);

  return (
    <>
      <h2>Count: {count}</h2>
      <h2>Square: {square}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>

      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
    </>
  );
}

export default App;
Console
Initial

Calculating...

Type in Input

(No Calculation)

Type Again

(No Calculation)

Click Increment

Calculating...

✅ The calculation only runs when count changes.

Example 3: Filtering a Large List
import React, { useMemo, useState } from "react";

function App() {
  const [search, setSearch] = useState("");

  const users = [
    "John",
    "Alice",
    "David",
    "Bob",
    "Charlie",
  ];

  const filteredUsers = useMemo(() => {
    console.log("Filtering...");
    return users.filter((user) =>
      user.toLowerCase().includes(search.toLowerCase())
    );
  }, [search]);

  return (
    <>
      <input
        placeholder="Search"
        value={search}
        onChange={(e) => setSearch(e.target.value)}
      />

      {filteredUsers.map((user) => (
        <p key={user}>{user}</p>
      ))}
    </>
  );
}
Output
Search: a

↓

Alice

David

Charlie
Example 4: Stable Object Reference

Without useMemo

const user = {
  name: "John",
};

<Child user={user} />;

Every render creates a new object.

With useMemo

const user = useMemo(() => ({
  name: "John",
}), []);

<Child user={user} />;

Now the object reference remains the same, helping React.memo() avoid unnecessary re-renders.

How useMemo() Works
Component Renders
        │
        ▼
Check Dependencies
        │
        ▼
Changed?
   /          \
 Yes          No
 │             │
Recalculate   Return Cached Value




Real-Time Use Cases
-----------------------------------------------------------
| Scenario            | Why use `useMemo()`?              |
| ------------------- | --------------------------------- |
| Filter users        | Avoid filtering every render      |
| Sort products       | Avoid repeated sorting            |
| Calculate totals    | Cache expensive calculations      |
| Dashboard charts    | Cache processed data              |
| Search results      | Cache filtered results            |
| Large tables        | Cache computed rows               |
| Stable object props | Prevent unnecessary child renders |


Benefits
✅ Improves performance
✅ Avoids unnecessary calculations
✅ Returns cached values
✅ Recalculates only when dependencies change
✅ Helps with React.memo()
✅ Useful for expensive computations
✅ Reduces CPU usage in large applications



Limitations
❌ Should not be used for every calculation.
❌ Memoization itself has a small overhead.
❌ Use it only when a calculation is expensive or a stable reference is needed.
❌ Incorrect dependency arrays can lead to stale values.

Interview Points
useMemo() is a React Hook used to memoize the result of a calculation.
It recalculates the value only when one of its dependencies changes.
It is mainly used for performance optimization.
It returns a cached value, not a function.
It is useful for expensive operations like filtering, sorting, and large computations.
It is commonly used to create stable object or array references for components wrapped with React.memo().
Unlike React.memo(), which memoizes components, useMemo() memoizes values.




7) useCallback() in React

useCallback() is a React Hook that memoizes (caches) a function. It returns the same function reference between renders unless one of its dependencies changes.

Simple Definition:

useCallback() prevents a function from being recreated on every render, which helps improve performance, especially when passing functions to child components wrapped with React.memo().

Syntax:
const memoizedFunction = useCallback(() => {
  // Function logic
}, [dependencies]);


Why do we use useCallback()? (7 Points)
1. Prevents Function Recreation ⭐

Normally, every render creates a new function.

Without useCallback

const handleClick = () => {
  console.log("Clicked");
};

Every render

Render 1

handleClick → Memory A

Render 2

handleClick → Memory B

Render 3

handleClick → Memory C

A new function is created each time.

2. Improves Performance ⭐

Reusing the same function reference avoids unnecessary work, especially in large applications.

3. Works Well with React.memo()

If a child component receives a function as a prop, it may re-render because the function reference changes.

useCallback() keeps the reference stable.

4. Prevents Unnecessary Child Re-renders

Without useCallback

Parent Re-render

↓

New Function Created

↓

Child Receives New Prop

↓

Child Re-renders

With useCallback

Parent Re-render

↓

Same Function Reference

↓

Child Props Same

↓

No Re-render
5. Memoizes Functions

It caches the function until one of its dependencies changes.

6. Recreates Function Only When Dependencies Change
const handleClick = useCallback(() => {
  console.log(count);
}, [count]);

The function is recreated only when count changes.

7. Useful in Large Applications

Common use cases:

Forms
Product List
Dashboard
Search
Tables
Chat Application
Example 1: Without useCallback()
Child Component
import React from "react";

function Child({ onClick }) {
  console.log("Child Rendered");

  return (
    <button onClick={onClick}>
      Child Button
    </button>
  );
}

export default React.memo(Child);
Parent Component
import React, { useState } from "react";
import Child from "./Child";

function App() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    console.log("Button Clicked");
  };

  return (
    <>
      <h2>{count}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>

      <Child onClick={handleClick} />
    </>
  );
}

export default App;
Console
Initial

Child Rendered

Click Increment

Child Rendered

Click Again

Child Rendered

❌ Every parent render creates a new handleClick, so the child re-renders.

Example 2: With useCallback()
Parent Component
import React, { useState, useCallback } from "react";
import Child from "./Child";

function App() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log("Button Clicked");
  }, []);

  return (
    <>
      <h2>{count}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>

      <Child onClick={handleClick} />
    </>
  );
}

export default App;
Console
Initial

Child Rendered

Click Increment

(No Child Render)

Click Again

(No Child Render)

✅ handleClick keeps the same reference, so React.memo() skips the child re-render.

Example 3: Dependency Array
import React, { useState, useCallback } from "react";

function App() {
  const [count, setCount] = useState(0);

  const showCount = useCallback(() => {
    console.log(count);
  }, [count]);

  return (
    <>
      <h2>{count}</h2>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>

      <button onClick={showCount}>
        Show Count
      </button>
    </>
  );
}

export default App;
Output
Initial

Count = 0

Click Show

0

Increment

Count = 1

Click Show

1

The function is recreated only when count changes.

Example 4: Real-Time Search
import React, { useCallback, useState } from "react";

function App() {
  const [search, setSearch] = useState("");

  const handleSearch = useCallback((e) => {
    setSearch(e.target.value);
  }, []);

  return (
    <>
      <input
        value={search}
        onChange={handleSearch}
      />

      <h2>{search}</h2>
    </>
  );
}
How useCallback() Works
Component Re-renders
        │
        ▼
Check Dependencies
        │
        ▼
Changed?
   /          \
 Yes          No
 │             │
Create New    Return Same
 Function     Function



Real-Time Use Cases

| Scenario              | Why use `useCallback()`?           |
| --------------------- | ---------------------------------- |
| Button click handlers | Stable function reference          |
| Form submission       | Avoid recreating handlers          |
| Search input          | Stable `onChange` handler          |
| Product list          | Prevent item re-renders            |
| Data tables           | Stable callbacks for rows          |
| Chat application      | Stable message handlers            |
| Dashboard             | Prevent unnecessary widget updates |

Example:

// useCallback
const handleClick = useCallback(() => {
  console.log("Clicked");
}, []);

// useMemo
const total = useMemo(() => price * quantity, [price, quantity]);

Benefits
✅ Prevents unnecessary function creation
✅ Improves performance
✅ Works well with React.memo()
✅ Keeps function references stable
✅ Prevents unnecessary child re-renders
✅ Useful in large applications
✅ Easy to optimize event handlers

Limitations
❌ Don't use it for every function.
❌ Memoization has a small overhead.
❌ Most beneficial when passing callbacks to memoized child components or when callbacks are dependencies of other hooks.
❌ Incorrect dependency arrays can cause stale closures or unexpected behavior.

Interview Points
useCallback() is a React Hook that memoizes a function.
It returns the same function reference unless its dependencies change.
It is mainly used for performance optimization.
It is commonly used with React.memo() to prevent unnecessary child re-renders.
It returns a function, whereas useMemo() returns a value.
It is useful when passing callbacks as props or when a callback is a dependency of useEffect or another hook.
Use useCallback() only when it provides a measurable benefit, not for every function.



8) forwardRef()
forwardRef() is a Higher-Order Component (HOC) that allows a parent component to pass a ref to a child component, so the parent can directly access the child's DOM element or imperative API.


Syntax
const Component = React.forwardRef((props, ref) => {
  return <input ref={ref} />;
});

or

import { forwardRef } from "react";

const Component = forwardRef((props, ref) => {
  return <input ref={ref} />;
});


Why do we use forwardRef()? (7 Points)
1. Pass a Ref from Parent to Child ⭐

Normally, a ref works only on DOM elements.

<input ref={inputRef} />

If you write:

<Child ref={inputRef} />

❌ It doesn't work for a regular functional component.

forwardRef() solves this problem.

2. Access Child DOM Elements ⭐

The parent can focus, scroll, or manipulate the child's DOM element.

Example:

inputRef.current.focus();
3. Reusable Components

Reusable UI components like:

Input
Button
TextArea
Modal

often use forwardRef().

4. Better Component Abstraction

The child component hides its implementation while still exposing the DOM element when needed.

5. Works with useRef()

forwardRef() is almost always used together with useRef().

const inputRef = useRef();

<CustomInput ref={inputRef} />
6. Used with useImperativeHandle()

You can expose only selected methods instead of the entire DOM element.

Example

focus()

clear()

scrollToTop()
7. Useful in Real Projects

Commonly used in:

Custom Input
Search Box
Modal
Dialog
Date Picker
Text Editor
UI Component Libraries
Example 1: Without forwardRef()
Child Component
function Child() {
  return <input />;
}

export default Child;
Parent Component
import React, { useRef } from "react";
import Child from "./Child";

function App() {
  const inputRef = useRef();

  return (
    <>
      <Child ref={inputRef} />
    </>
  );
}
Result
❌ Error

Function components cannot be given refs.

Reason:

Parent

↓

Child Component

↓

React doesn't know
which DOM element
the ref should point to.
Example 2: With forwardRef()
Child Component
import React, { forwardRef } from "react";

const Child = forwardRef((props, ref) => {
  return (
    <input
      ref={ref}
      placeholder="Enter Name"
    />
  );
});

export default Child;
Parent Component
import React, { useRef } from "react";
import Child from "./Child";

function App() {
  const inputRef = useRef();

  const focusInput = () => {
    inputRef.current.focus();
  };

  return (
    <>
      <Child ref={inputRef} />

      <button onClick={focusInput}>
        Focus Input
      </button>
    </>
  );
}

export default App;
Output
Input

___________

[Focus Input]

↓

Click Button

↓

Cursor moves inside input
Example 3: Scroll to Child Element
Child
import React, { forwardRef } from "react";

const Child = forwardRef((props, ref) => {
  return (
    <div
      ref={ref}
      style={{
        marginTop: "800px",
      }}
    >
      Bottom Section
    </div>
  );
});

export default Child;
Parent
import React, { useRef } from "react";
import Child from "./Child";

function App() {
  const sectionRef = useRef();

  return (
    <>
      <button
        onClick={() =>
          sectionRef.current.scrollIntoView({
            behavior: "smooth",
          })
        }
      >
        Scroll
      </button>

      <Child ref={sectionRef} />
    </>
  );
}
Example 4: forwardRef() + useImperativeHandle()
Child
import React, {
  forwardRef,
  useImperativeHandle,
  useRef,
} from "react";

const Child = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },

    clear: () => {
      inputRef.current.value = "";
    },
  }));

  return <input ref={inputRef} />;
});

export default Child;
Parent
import React, { useRef } from "react";
import Child from "./Child";

function App() {
  const childRef = useRef();

  return (
    <>
      <Child ref={childRef} />

      <button
        onClick={() => childRef.current.focus()}
      >
        Focus
      </button>

      <button
        onClick={() => childRef.current.clear()}
      >
        Clear
      </button>
    </>
  );
}
Output
Click Focus

↓

Cursor moves to input

Click Clear

↓

Input becomes empty
How forwardRef() Works
Parent Component
      │
      │ ref
      ▼
Child Component (forwardRef)
      │
      ▼
<input />

Parent can now access

inputRef.current

Real-Time Use Cases
| Component                             | Why use `forwardRef()`?               |
| ------------------------------------- | ------------------------------------- |
| Custom Input                          | Focus input from parent               |
| Search Box                            | Auto-focus search field               |
| Modal                                 | Focus first input when opened         |
| TextArea                              | Clear content from parent             |
| Date Picker                           | Open calendar programmatically        |
| Rich Text Editor                      | Control editor methods                |
| UI Libraries (Material UI, Chakra UI) | Pass refs through reusable components |


Benefits
✅ Parent can access child DOM elements.
✅ Enables reusable components to support refs.
✅ Works well with useRef().
✅ Can be combined with useImperativeHandle().
✅ Helps create flexible UI component libraries.
✅ Supports focus, scrolling, and other imperative DOM operations.
✅ Improves encapsulation by exposing only what the parent needs.


Interview Points
forwardRef() allows a parent component to pass a ref to a child functional component.
It is commonly used with useRef().
It is useful for accessing child DOM elements like inputs or buttons.
It is often combined with useImperativeHandle() to expose selected methods instead of the entire DOM element.
It is widely used in reusable UI components and component libraries.
Without forwardRef(), a regular functional component cannot receive a ref prop.
React 19 note: React 19 introduces support for passing ref as a normal prop in many cases, reducing the need for forwardRef() in new code. However, forwardRef() is still very common in existing codebases and is an important interview topic because many applications are built with React 16–18.


9) useImperativeHandle()


Instead of exposing the entire DOM element, you can expose only specific methods or properties.

Simple Definition:

useImperativeHandle() lets the child component decide what the parent can access through a ref.


Syntax
useImperativeHandle(ref, () => ({
  method1() {},
  method2() {},
}), [dependencies]);


Why do we use useImperativeHandle()? (7 Points)
1. Expose Only Required Methods ⭐

Instead of exposing the whole input element:

inputRef.current

You expose only:

focus()

clear()

This improves encapsulation.

2. Provides Better Encapsulation ⭐

The parent doesn't know how the child works internally.

Only selected methods are available.

Parent

↓

focus()

clear()

↓

Child handles everything internally
3. Works with forwardRef()

useImperativeHandle() must be used together with forwardRef().

const Child = forwardRef((props, ref) => {})
4. Hide Internal DOM Elements

Without useImperativeHandle()

childRef.current.focus()
childRef.current.value
childRef.current.style

Parent gets complete access.

With useImperativeHandle()

childRef.current.focus()

childRef.current.clear()

Only these methods are available.

5. Create Custom APIs

You can expose your own methods.

Example

openModal()

closeModal()

resetForm()

playVideo()
6. Improves Component Reusability

Reusable components become cleaner because parents interact with a simple API instead of internal implementation details.

7. Useful in Large Applications

Common use cases:

Modal
Form
Input
Video Player
Rich Text Editor
Date Picker
Carousel

Example 1: Focus and Clear Input
Child Component
import React, {
  forwardRef,
  useRef,
  useImperativeHandle,
} from "react";

const Child = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current.focus();
    },

    clear() {
      inputRef.current.value = "";
    },
  }));

  return (
    <input
      ref={inputRef}
      placeholder="Enter Name"
    />
  );
});

export default Child;
Parent Component
import React, { useRef } from "react";
import Child from "./Child";

function App() {
  const childRef = useRef();

  return (
    <>
      <Child ref={childRef} />

      <button
        onClick={() => childRef.current.focus()}
      >
        Focus
      </button>

      <button
        onClick={() => childRef.current.clear()}
      >
        Clear
      </button>
    </>
  );
}

export default App;
Output
Input

_____________

[Focus]

[Clear]

Click Focus

↓

Cursor moves to input

Click Clear

↓

Input becomes empty
Example 2: Modal Open and Close
Child Component
import React, {
  forwardRef,
  useImperativeHandle,
  useState,
} from "react";

const Modal = forwardRef((props, ref) => {
  const [open, setOpen] = useState(false);

  useImperativeHandle(ref, () => ({
    openModal() {
      setOpen(true);
    },

    closeModal() {
      setOpen(false);
    },
  }));

  return (
    <>
      {open && (
        <div
          style={{
            border: "1px solid black",
            padding: "20px",
          }}
        >
          <h2>Modal Open</h2>
        </div>
      )}
    </>
  );
});

export default Modal;
Parent Component
import React, { useRef } from "react";
import Modal from "./Modal";

function App() {
  const modalRef = useRef();

  return (
    <>
      <button
        onClick={() =>
          modalRef.current.openModal()
        }
      >
        Open
      </button>

      <button
        onClick={() =>
          modalRef.current.closeModal()
        }
      >
        Close
      </button>

      <Modal ref={modalRef} />
    </>
  );
}

export default App;
Output
Click Open

↓

Modal Opens

Click Close

↓

Modal Closes
Example 3: Video Player
Child
import React, {
  forwardRef,
  useImperativeHandle,
  useRef,
} from "react";

const VideoPlayer = forwardRef((props, ref) => {
  const videoRef = useRef();

  useImperativeHandle(ref, () => ({
    play() {
      videoRef.current.play();
    },

    pause() {
      videoRef.current.pause();
    },
  }));

  return (
    <video
      ref={videoRef}
      width="300"
      src="video.mp4"
    />
  );
});

export default VideoPlayer;

Parent
import React, { useRef } from "react";
import VideoPlayer from "./VideoPlayer";

function App() {
  const videoRef = useRef();

  return (
    <>
      <VideoPlayer ref={videoRef} />

      <button
        onClick={() => videoRef.current.play()}
      >
        Play
      </button>

      <button
        onClick={() => videoRef.current.pause()}
      >
        Pause
      </button>
    </>
  );
}
How useImperativeHandle() Works
Parent Component
        │
        │ ref
        ▼
Child Component
        │
        ▼
useImperativeHandle()

        │
        ▼

{
   focus(),
   clear()
}

Parent can access only

focus()

clear()
Without useImperativeHandle()
Parent

↓

inputRef.current

↓

Entire DOM Element

focus()

value

style

className

everything...
With useImperativeHandle()
Parent

↓

childRef.current

↓

focus()

clear()

Only these methods

Real-Time Use Cases

| Component        | Exposed Methods               |
| ---------------- | ----------------------------- |
| Input            | `focus()`, `clear()`          |
| Modal            | `openModal()`, `closeModal()` |
| Video Player     | `play()`, `pause()`           |
| Audio Player     | `play()`, `stop()`            |
| Carousel         | `next()`, `previous()`        |
| Form             | `reset()`, `validate()`       |
| Rich Text Editor | `undo()`, `redo()`, `clear()` |


Benefits
✅ Exposes only selected methods.
✅ Hides internal implementation details.
✅ Improves component encapsulation.
✅ Creates a clean API for parent components.
✅ Works with reusable components.
✅ Often used in UI libraries.
✅ Makes components easier to maintain.


Interview Points
useImperativeHandle() customizes the value exposed through a ref.
It is used together with forwardRef().
It allows the child component to expose only selected methods or properties.
It helps hide internal DOM elements and implementation details.
Common examples include exposing methods like focus(), clear(), openModal(), and closeModal().
It improves encapsulation and makes reusable components easier to use.
Use it only when imperative control is necessary; for most parent-child communication, prefer passing props and callbacks.


10) createPortal()

What is a Portal?

A Portal allows you to render a React component into a different DOM node while keeping it part of the same React component tree.

For example:

Normally

<div id="root">
    App
      ├── Header
      ├── Main
      └── Modal
</div>

Using Portal

<div id="root">
    App
      ├── Header
      └── Main
</div>

<div id="modal-root">
      Modal
</div>

Although the Modal is rendered outside #root, it still behaves like a child of App.

Syntax
import { createPortal } from "react-dom";

createPortal(
  component,
  document.getElementById("modal-root")
);


Why do we use createPortal()? (7 Points)
1. Render Outside Parent DOM ⭐

A component can be rendered outside its parent's HTML structure.

Example:

App

↓

Modal

↓

Rendered in

modal-root
2. Avoid CSS Overflow Issues ⭐

Suppose your parent has

overflow: hidden;

Without Portal

Modal

↓

Gets clipped ❌

With Portal

Modal

↓

Appears above everything ✅
3. Perfect for Modals

Most applications render dialogs using portals.

Examples:

Login Modal
Delete Confirmation
Payment Popup
4. Great for Tooltips

Tooltips should appear above all elements.

Instead of being hidden by parent containers, render them using a portal.

5. Useful for Dropdowns

Dropdown menus inside tables or cards may be cut off.

Portal solves this issue.

6. Maintains React Tree

Even though the HTML is rendered elsewhere:

Context works ✅
Props work ✅
State works ✅
Event bubbling still works in React ✅
7. Better User Experience

Common components using portals:

Modal
Tooltip
Toast
Popover
Sidebar Overlay
Loading Overlay
Context Menu
Example 1: Modal using createPortal()
Step 1: index.html
<body>
  <div id="root"></div>

  <div id="modal-root"></div>
</body>
Step 2: Modal Component
import { createPortal } from "react-dom";

function Modal({ onClose }) {
  return createPortal(
    <div
      style={{
        position: "fixed",
        inset: 0,
        background: "rgba(0,0,0,0.5)",
      }}
    >
      <div
        style={{
          background: "white",
          width: "300px",
          margin: "100px auto",
          padding: "20px",
        }}
      >
        <h2>React Portal</h2>

        <button onClick={onClose}>
          Close
        </button>
      </div>
    </div>,
    document.getElementById("modal-root")
  );
}

export default Modal;
Step 3: App Component
import React, { useState } from "react";
import Modal from "./Modal";

function App() {
  const [show, setShow] = useState(false);

  return (
    <>
      <button onClick={() => setShow(true)}>
        Open Modal
      </button>

      {show && (
        <Modal onClose={() => setShow(false)} />
      )}
    </>
  );
}

export default App;
Output
[Open Modal]

↓

Click

↓

Modal Opens

↓

Click Close

↓

Modal Closes
Example 2: Tooltip using createPortal()
import { createPortal } from "react-dom";

function Tooltip() {
  return createPortal(
    <div
      style={{
        position: "fixed",
        top: 20,
        right: 20,
        background: "black",
        color: "white",
        padding: "10px",
      }}
    >
      Hello Tooltip
    </div>,
    document.body
  );
}

export default Tooltip;

The tooltip is rendered directly inside <body>, so it isn't affected by parent containers.

How createPortal() Works
React Component
        │
        ▼
createPortal()
        │
        ▼
Another DOM Node

modal-root

or

document.body

Real-Time Use Cases
| Component          | Why use Portal?                       |
| ------------------ | ------------------------------------- |
| Modal              | Display above the entire page         |
| Tooltip            | Avoid clipping by parent containers   |
| Dropdown           | Show outside overflow-hidden elements |
| Toast Notification | Display globally                      |
| Context Menu       | Position freely on the page           |
| Loading Overlay    | Cover the whole screen                |
| Sidebar Overlay    | Render above all content              |



Benefits
✅ Renders outside the parent DOM hierarchy.
✅ Solves overflow: hidden and z-index issues.
✅ Perfect for modals, tooltips, and dropdowns.
✅ Preserves React state, props, and context.
✅ Supports React event bubbling.
✅ Improves user experience.
✅ Easy to integrate into existing applications.


Interview Points
There is no usePortal() hook in React; the correct API is createPortal() from react-dom.
createPortal() renders a React element into a different DOM node.
It is commonly used for modals, tooltips, dropdowns, and overlays.
Even though the DOM location changes, the component remains part of the same React tree.
React Context, state, and props continue to work normally through portals.
React events still bubble through the React component tree, not just the DOM tree.
Portals are mainly used to avoid layout problems such as overflow: hidden and stacking (z-index) issues.

12) useLayoutEffect()

useLayoutEffect() is a React Hook that runs synchronously after React updates the DOM but before the browser paints (displays) the screen.

Simple Definition:

useLayoutEffect() is used when you need to measure or modify the DOM before the user sees it, preventing visual flickering.

Syntax:
useLayoutEffect(() => {
  // DOM measurement or manipulation

  return () => {
    // Cleanup
  };
}, [dependencies]);


Why do we use useLayoutEffect()? (7 Points)
1. Runs Before Browser Paint ⭐

Execution order:

Render Component

↓

Update DOM

↓

useLayoutEffect()

↓

Browser Paint (Screen Update)

This ensures users only see the final UI.

2. Prevents UI Flickering ⭐

Suppose you need to change an element's size or position.

Without useLayoutEffect

Render

↓

User sees incorrect UI

↓

Effect runs

↓

UI changes

👎 Users notice a flicker.

With useLayoutEffect

Render

↓

Effect runs

↓

Browser paints

↓

User sees correct UI

👍 No flicker.

3. Measure DOM Elements

Useful for reading:

Width
Height
Position
Scroll position

Example:

const width = divRef.current.offsetWidth;
4. Manipulate DOM Before Paint

Examples:

Focus input
Scroll element
Set element size
Update styles
5. Works with useRef()

Usually used together.

const divRef = useRef();

useLayoutEffect(() => {
  console.log(divRef.current.offsetWidth);
}, []);
6. Cleanup Support

Like useEffect, it supports cleanup.

useLayoutEffect(() => {

   return () => {
      console.log("Cleanup");
   }

}, []);
7. Used for UI Calculations

Real projects:

Tooltip positioning
Modal positioning
Auto scroll
Drag & Drop
Animations
Responsive layouts
Example 1: Measure Element Width
import React, { useRef, useLayoutEffect } from "react";

function App() {
  const boxRef = useRef();

  useLayoutEffect(() => {
    console.log(
      "Width:",
      boxRef.current.offsetWidth
    );
  }, []);

  return (
    <div
      ref={boxRef}
      style={{
        width: "300px",
        background: "lightblue",
      }}
    >
      Hello React
    </div>
  );
}

export default App;
Output
Width: 300
Example 2: Auto Focus Input
import React, {
  useRef,
  useLayoutEffect,
} from "react";

function App() {
  const inputRef = useRef();

  useLayoutEffect(() => {
    inputRef.current.focus();
  }, []);

  return (
    <input
      ref={inputRef}
      placeholder="Enter Name"
    />
  );
}

export default App;
Output
Page Opens

↓

Cursor automatically appears
inside the input

Because focus happens before the browser paints.

Example 3: Scroll to Bottom
import React, {
  useRef,
  useLayoutEffect,
} from "react";

function App() {
  const bottomRef = useRef();

  useLayoutEffect(() => {
    bottomRef.current.scrollIntoView();
  }, []);

  return (
    <>
      <div style={{ height: "1000px" }}></div>

      <h2 ref={bottomRef}>
        Bottom
      </h2>
    </>
  );
}

export default App;

When the page loads, it scrolls directly to the bottom before the user sees the initial position.

Example 4: Change Style Before Paint
import React, {
  useLayoutEffect,
  useRef,
} from "react";

function App() {
  const boxRef = useRef();

  useLayoutEffect(() => {
    boxRef.current.style.background =
      "green";
  }, []);

  return (
    <div
      ref={boxRef}
      style={{
        width: "200px",
        height: "100px",
        background: "red",
      }}
    >
      Box
    </div>
  );
}

export default App;
Output
User immediately sees

Green Box

(No Red Flash)

useEffect() vs useLayoutEffect()
| Feature         | `useEffect()`                    | `useLayoutEffect()`                 |
| --------------- | -------------------------------- | ----------------------------------- |
| Runs            | After browser paint              | Before browser paint                |
| Blocks painting | ❌ No                             | ✅ Yes                               |
| Used for        | API calls, timers, subscriptions | DOM measurements and layout changes |
| UI flicker      | May happen                       | Prevented                           |
| Performance     | Better for most side effects     | Use only when necessary             |



Execution Flow
useEffect()
Render

↓

DOM Updated

↓

Browser Paint

↓

useEffect()
useLayoutEffect()
Render

↓

DOM Updated

↓

useLayoutEffect()

↓

Browser Paint


Real-Time Use Cases

| Scenario             | Why use `useLayoutEffect()`?           |
| -------------------- | -------------------------------------- |
| Measure element size | Get accurate width/height before paint |
| Auto focus input     | Focus before the screen is shown       |
| Tooltip positioning  | Calculate position without flicker     |
| Modal positioning    | Place modal correctly before paint     |
| Auto scroll          | Scroll before user sees the page       |
| Animations           | Apply initial styles before paint      |
| Drag & Drop          | Measure layout before interaction      |


Benefits
✅ Runs before browser paint.
✅ Prevents UI flickering.
✅ Ideal for measuring DOM elements.
✅ Supports cleanup.
✅ Works well with useRef().
✅ Helps build smooth UI interactions.
✅ Useful for layout calculations.

Limitations
❌ Blocks the browser from painting while it runs.
❌ Overusing it can hurt performance.
❌ Not suitable for API calls or other asynchronous side effects.
❌ Prefer useEffect() unless you specifically need synchronous DOM measurement or updates.

Interview Points
useLayoutEffect() runs synchronously after the DOM is updated but before the browser paints.
It is mainly used for DOM measurements and layout-related updates.
It helps prevent visual flickering by applying changes before the user sees the UI.
It is commonly used with useRef() to access DOM elements.
It supports cleanup just like useEffect().
It should be used sparingly because it blocks browser painting.
Use useEffect() for most side effects, and choose useLayoutEffect() only when you need to read or modify the DOM before paint.




12) useReducer()
useReducer() is a React Hook used to manage complex state logic. It is an alternative to useState() when state updates depend on the previous state or when you have multiple related state values.

Simple Definition:

useReducer() manages state using a reducer function and actions. Instead of updating state directly, you dispatch an action, and the reducer decides how the state should change.

Syntax
const [state, dispatch] = useReducer(reducer, initialState);
Parameters
state → Current state
dispatch → Function to send an action
reducer → Function that updates the state
initialState → Initial state value



How useReducer() Works
Button Click

↓

dispatch({ type: "increment" })

↓

Reducer Function

↓

New State

↓

Component Re-renders
Why do we use useReducer()? (7 Points)
1. Handles Complex State ⭐

When multiple state values are related, useReducer() is easier to manage than multiple useState() calls.

Example:

const initialState = {
  count: 0,
  loading: false,
  error: "",
};
2. Centralizes State Logic ⭐

Instead of writing update logic in multiple places, all state updates are inside the reducer.

3. Easier to Manage Multiple Actions

You can define actions like:

INCREMENT

DECREMENT

RESET

Each action has its own logic.

4. Predictable State Updates

Every state update follows the same flow:

dispatch()

↓

Reducer

↓

New State

This makes debugging easier.

5. Better for Large Applications

Useful for:

Forms
Shopping Cart
Authentication
Dashboard
Todo List
API State
6. Similar to Redux

useReducer() follows the same concepts as Redux:

State
Action
Reducer
Dispatch

So learning useReducer() helps in understanding Redux.

7. Improves Code Organization

Business logic stays inside the reducer, while the component focuses on rendering the UI.

Example 1: Counter
import React, { useReducer } from "react";

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return {
        count: state.count + 1,
      };

    case "decrement":
      return {
        count: state.count - 1,
      };

    case "reset":
      return initialState;

    default:
      return state;
  }
}

function App() {
  const [state, dispatch] = useReducer(
    reducer,
    initialState
  );

  return (
    <div>
      <h2>Count: {state.count}</h2>

      <button
        onClick={() =>
          dispatch({ type: "increment" })
        }
      >
        +
      </button>

      <button
        onClick={() =>
          dispatch({ type: "decrement" })
        }
      >
        -
      </button>

      <button
        onClick={() =>
          dispatch({ type: "reset" })
        }
      >
        Reset
      </button>
    </div>
  );
}

export default App;
Output
Initial

Count : 0

Click +

Count : 1

Click +

Count : 2

Click -

Count : 1

Click Reset

Count : 0
Example 2: Login Form
import React, { useReducer } from "react";

const initialState = {
  email: "",
  password: "",
};

function reducer(state, action) {
  switch (action.type) {
    case "SET_EMAIL":
      return {
        ...state,
        email: action.payload,
      };

    case "SET_PASSWORD":
      return {
        ...state,
        password: action.payload,
      };

    case "RESET":
      return initialState;

    default:
      return state;
  }
}

function App() {
  const [state, dispatch] = useReducer(
    reducer,
    initialState
  );

  return (
    <div>
      <input
        type="email"
        placeholder="Email"
        value={state.email}
        onChange={(e) =>
          dispatch({
            type: "SET_EMAIL",
            payload: e.target.value,
          })
        }
      />

      <input
        type="password"
        placeholder="Password"
        value={state.password}
        onChange={(e) =>
          dispatch({
            type: "SET_PASSWORD",
            payload: e.target.value,
          })
        }
      />

      <button
        onClick={() =>
          dispatch({ type: "RESET" })
        }
      >
        Reset
      </button>

      <h3>Email: {state.email}</h3>
      <h3>Password: {state.password}</h3>
    </div>
  );
}

export default App;
Output
Email : abc@gmail.com

Password : 123456

Click Reset

↓

Email :

Password :
Example 3: Shopping Cart
import React, { useReducer } from "react";

const initialState = {
  cart: [],
};

function reducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM":
      return {
        cart: [...state.cart, action.payload],
      };

    default:
      return state;
  }
}

function App() {
  const [state, dispatch] = useReducer(
    reducer,
    initialState
  );

  return (
    <>
      <button
        onClick={() =>
          dispatch({
            type: "ADD_ITEM",
            payload: "Laptop",
          })
        }
      >
        Add Laptop
      </button>

      {state.cart.map((item, index) => (
        <p key={index}>{item}</p>
      ))}
    </>
  );
}
How useReducer() Works Internally
Initial State

↓

{
   count: 0
}

↓

dispatch({type:"increment"})

↓

Reducer

↓

{
   count:1
}

↓

Re-render


useState() vs useReducer()
| Feature                 | `useState()`     | `useReducer()`   |
| ----------------------- | ---------------- | ---------------- |
| Best for                | Simple state     | Complex state    |
| State updates           | Directly         | Through reducer  |
| Multiple related states | Difficult        | Easy             |
| Uses actions            | ❌                | ✅                |
| Scales well             | Small components | Large components |


Real-Time Use Cases
| Scenario        | Why use `useReducer()`?               |
| --------------- | ------------------------------------- |
| Login Form      | Manage multiple input fields          |
| Shopping Cart   | Add, remove, update items             |
| Todo App        | Add, edit, delete tasks               |
| Authentication  | Login, logout, refresh user           |
| Dashboard       | Handle multiple related state values  |
| API Requests    | Loading, success, error states        |
| Multi-step Form | Manage step transitions and form data |

Benefits
✅ Handles complex state easily.
✅ Keeps state logic in one place.
✅ Easy to debug.
✅ Similar pattern to Redux.
✅ Better organization.
✅ Predictable state updates.
✅ Scales well in larger components.
Limitations
❌ More boilerplate than useState().
❌ Not necessary for simple state like a single counter or input.
❌ Reducers should remain pure (no API calls or side effects inside the reducer).
Interview Points
useReducer() is a React Hook for managing complex state.
It returns an array: [state, dispatch].
State updates are handled by a reducer function based on dispatched actions.
It is useful when state updates depend on previous state or when multiple state values are related.
It follows the same concepts as Redux: state → action → reducer → dispatch.
Reducers should be pure functions: they should return new state without mutating the existing state or performing side effects.
Use useState() for simple state and useReducer() for more complex state management.
Easy Way to Remember
useState()
Button Click

↓

setCount(count + 1)

↓

State Updated
useReducer()
Button Click

↓

dispatch({ type: "increment" })

↓

Reducer()

↓

New State

↓

UI Updates
Interview Rule
useState → Simple state (counter, input field).
useReducer → Complex state (forms, shopping cart, authentication, dashboards).


13) createContext(), Provider, and useContext() in React

React Context is used to share data between components without passing props manually at every level (avoiding prop drilling).


What is Prop Drilling?

Suppose you have the following component tree:

App
 │
 ├── Parent
 │      │
 │      ├── Child
 │      │      │
 │      │      └── GrandChild

If App has user data:

const user = "John";

Without Context

App

↓

Parent

↓

Child

↓

GrandChild

You must pass props through every component:

<App user={user} />

↓

<Parent user={user} />

↓

<Child user={user} />

↓

<GrandChild user={user} />

Even though only GrandChild needs the data.

This is called Prop Drilling.

React Context Solution

React Context allows components to access shared data directly.

App

↓

Context Provider

↓

GrandChild

(No Prop Drilling)
Three Steps of Context API
1. createContext()

↓

2. Provider

↓

3. useContext()
1. createContext()
Definition

createContext() creates a Context object.

Syntax
const UserContext = createContext();
Why do we use createContext()? (7 Points)
1. Creates a Context Object ⭐

It creates a place where data can be stored and shared.

2. Avoids Prop Drilling ⭐

No need to pass props through every component.

3. Share Global Data

Examples:

User
Theme
Language
Authentication
Currency
4. Works with Provider

The Provider supplies the value.

5. Works with useContext

Components consume the value using useContext().

6. One Context, Many Components

Many components can access the same data.

7. Better Code Organization

Global data is managed in one place.

Example
import { createContext } from "react";

export const UserContext = createContext();
2. Provider
Definition

A Provider supplies data to all child components.

Syntax
<UserContext.Provider value={value}>
    <App />
</UserContext.Provider>
Why do we use Provider? (7 Points)
1. Shares Data ⭐

Provides data to all child components.

2. Eliminates Prop Drilling

No manual prop passing.

3. Updates All Consumers

Changing the Provider's value updates all consuming components.

4. Global State

Ideal for:

Theme
Login
Language
5. Easy to Maintain

One update affects all consumers.

6. Can Wrap Entire App

Usually placed around the root component.

7. Multiple Providers Supported

You can use:

ThemeProvider
AuthProvider
LanguageProvider

together.

Example
import React, { createContext } from "react";

export const UserContext = createContext();

function App() {
  return (
    <UserContext.Provider value="John">
      <Home />
    </UserContext.Provider>
  );
}
3. useContext()
Definition

useContext() reads the value from the nearest matching Provider.

Syntax
const value = useContext(UserContext);
Why do we use useContext()? (7 Points)
1. Read Context Value ⭐

Access shared data.

2. Avoid Prop Drilling ⭐

No props required.

3. Easy to Use

One line:

const user = useContext(UserContext);
4. Access Anywhere

Any child component inside the Provider can read the value.

5. Automatically Updates

If Provider value changes, consumers re-render.

6. Cleaner Components

No unnecessary props.

7. Works with Multiple Contexts
const user = useContext(UserContext);
const theme = useContext(ThemeContext);
Complete Example
Step 1: Create Context
UserContext.js
import { createContext } from "react";

export const UserContext = createContext();
Step 2: Provider
App.jsx
import React from "react";
import { UserContext } from "./UserContext";
import Home from "./Home";

function App() {
  return (
    <UserContext.Provider value="John">
      <Home />
    </UserContext.Provider>
  );
}

export default App;
Step 3: Parent Component
import Child from "./Child";

function Home() {
  return <Child />;
}

export default Home;
Step 4: Child Component
import GrandChild from "./GrandChild";

function Child() {
  return <GrandChild />;
}

export default Child;
Step 5: GrandChild Component
import React, { useContext } from "react";
import { UserContext } from "./UserContext";

function GrandChild() {
  const user = useContext(UserContext);

  return <h2>Hello {user}</h2>;
}

export default GrandChild;
Output
Hello John

Notice:

App

↓

Home

↓

Child

↓

GrandChild

↓

useContext()

↓

John

No props were passed.

Example 2: Theme Switcher
import React, {
  createContext,
  useContext,
  useState,
} from "react";

const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext.Provider
      value={{ theme, setTheme }}
    >
      <Home />
    </ThemeContext.Provider>
  );
}

function Home() {
  return <Button />;
}

function Button() {
  const { theme, setTheme } =
    useContext(ThemeContext);

  return (
    <>
      <h2>{theme}</h2>

      <button
        onClick={() =>
          setTheme(
            theme === "light"
              ? "dark"
              : "light"
          )
        }
      >
        Toggle Theme
      </button>
    </>
  );
}

export default App;
Output
Light

↓

Click Button

↓

Dark

↓

Click Again

↓

Light
How Context API Works
createContext()

↓

Provider

↓

value

↓

useContext()

↓

Component Gets Data



Real-Time Use Cases

| Context             | Used For             |
| ------------------- | -------------------- |
| AuthContext         | Logged-in user       |
| ThemeContext        | Dark/Light theme     |
| LanguageContext     | English/Hindi        |
| CartContext         | Shopping cart        |
| UserContext         | User profile         |
| NotificationContext | Toast messages       |
| SettingsContext     | Application settings |


useContext() vs Props
| Props               | Context                     |
| ------------------- | --------------------------- |
| Passed manually     | Accessed directly           |
| Prop drilling       | No prop drilling            |
| Best for local data | Best for shared/global data |


Benefits
✅ Eliminates prop drilling.
✅ Shares data globally.
✅ Cleaner component hierarchy.
✅ Easy to maintain.
✅ Automatically updates consumers.
✅ Supports multiple contexts.
✅ Built into React (no external library required).

Limitations
❌ Every consumer re-renders when the Provider's value changes.
❌ Not intended as a replacement for all state management solutions.
❌ Frequently changing data can affect performance if too many components consume the same context.

Interview Points
createContext() creates a Context object.
A Provider makes a value available to all descendant components.
useContext() reads the value from the nearest matching Provider.
The Context API helps avoid prop drilling.
It is commonly used for shared data such as authentication, themes, languages, and user information.
When the Provider's value changes, all consuming components re-render with the new value.
For highly complex or frequently changing global state, Context is often combined with patterns like reducers or dedicated state management libraries.

