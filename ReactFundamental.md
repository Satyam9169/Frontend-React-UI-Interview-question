# React Fundamentals Interview Questions (Module 1)

> React Interview Preparation (2–4 Years Experience)

---

# 1. What is React?

## Answer

React is an open-source JavaScript library developed by Facebook (Meta) for building fast and interactive User Interfaces (UI), especially Single Page Applications (SPA).

React allows developers to build reusable UI components and efficiently update only the changed parts of the UI instead of reloading the entire page.

### Example

```jsx
function App() {
  return <h1>Hello React</h1>;
}
```

---

# 2. Why was React created?

## Answer

Before React, updating the UI manually using JavaScript or jQuery became difficult as applications grew larger.

Facebook created React in 2013 to solve problems related to:

- Complex UI updates
- Poor performance
- Difficult code maintenance
- Reusability issues

React introduced:

- Component-based architecture
- Virtual DOM
- One-way data flow

---

# 3. What problem does React solve?

## Answer

React solves several frontend development problems:

### 1. UI becomes difficult to manage

Without React:

```javascript
document.getElementById("title").innerHTML = "Hello";
```

Developers manually update DOM.

With React:

```jsx
setTitle("Hello");
```

React updates UI automatically.

---

### 2. Code Reusability

React components can be reused multiple times.

```jsx
<Button />
<Button />
<Button />
```

---

### 3. Performance

React updates only changed elements using Virtual DOM.

---

### 4. Maintainability

Large applications become easier to manage because everything is divided into components.

---

# 4. What are the features of React?

## Answer

Major features are:

- Component-Based Architecture
- Virtual DOM
- JSX
- One-Way Data Binding
- Reusable Components
- Declarative Programming
- Hooks
- Fast Rendering
- React Developer Tools
- Strong Community Support

---

# 5. What are the advantages of React?

## Answer

- Easy to learn
- Reusable components
- Better performance
- Virtual DOM
- Large ecosystem
- Easy debugging
- SEO support (using Next.js)
- Huge community support
- Declarative programming
- One-way data flow

---

# 6. What are the limitations of React?

## Answer

- Only handles UI layer
- Frequent updates
- JSX may look unfamiliar
- Requires additional libraries
- State management needed for large apps
- Learning curve for Hooks, Context, Redux

---

# 7. What is SPA?

## Answer

SPA stands for **Single Page Application**.

Only one HTML page is loaded.

Whenever users navigate between pages, React updates only the required components instead of loading an entirely new page.

Examples:

- Gmail
- Facebook
- Instagram
- Twitter

---

# 8. Difference between SPA and MPA

| SPA | MPA |
|------|------|
| Single HTML page | Multiple HTML pages |
| Faster navigation | Slower navigation |
| Uses JavaScript | Server renders every page |
| Better User Experience | Page reloads every request |
| React, Angular, Vue | PHP, ASP.NET, JSP |

---

# 9. Why is React called a library and not a framework?

## Answer

React is called a library because it focuses only on building the User Interface.

It does not include:

- Routing
- State management
- HTTP requests
- Form handling

Developers choose additional libraries like:

- React Router
- Redux Toolkit
- Axios

Frameworks like Angular provide everything in one package.

---

# 10. How does React work internally?

## Answer

React follows these steps:

```
State Changes

↓

Component Re-renders

↓

Creates New Virtual DOM

↓

Compares with Old Virtual DOM

↓

Finds Differences

↓

Updates Real DOM

↓

Browser Paints UI
```

React updates only changed nodes instead of rebuilding the entire page.

---

# 11. Explain React Architecture.

## Answer

React follows Component-Based Architecture.

```
App

├── Navbar

├── Sidebar

├── ProductList
│     ├── ProductCard
│     ├── ProductCard
│     └── ProductCard

└── Footer
```

Each component has:

- Own UI
- Own logic
- Own state
- Reusable behavior

---

# 12. What is Virtual DOM and how does it work internally?

## Answer

Virtual DOM is a lightweight JavaScript representation of the Real DOM.

Example

Real DOM

```html
<h1>Hello</h1>
```

Virtual DOM

```javascript
{
 type: "h1",
 props: {
   children: "Hello"
 }
}
```

---

## Internal Working

Step 1

Initial Render

```
JSX

↓

Virtual DOM

↓

Real DOM
```

---

Step 2

State Changes

```
setState()

↓

New Virtual DOM
```

---

Step 3

React compares

Old Virtual DOM

VS

New Virtual DOM

---

Step 4

Diffing Algorithm identifies changes.

---

Step 5

Only changed DOM nodes are updated.

---

# 13. What is Real DOM?

## Answer

Real DOM is the actual HTML structure created by the browser.

Example

```html
<div>
   <h1>Hello</h1>
</div>
```

Every DOM update causes:

- Layout
- Repaint
- Reflow

These operations are expensive.

---

# 14. Difference between Virtual DOM and Real DOM

| Virtual DOM | Real DOM |
|-------------|----------|
| JavaScript Object | Actual Browser DOM |
| Lightweight | Heavy |
| Fast | Slower |
| Updates only changed nodes | May update entire tree |
| Exists in memory | Exists in browser |

---

# 15. How does React update the DOM?

## Answer

Flow

```
User Action

↓

State Changes

↓

Component Re-render

↓

Creates New Virtual DOM

↓

Diffing

↓

Reconciliation

↓

Update Changed Nodes

↓

Real DOM Updated
```

React never directly manipulates the DOM first.

It first updates Virtual DOM.

---

# 16. What is Reconciliation?

## Answer

Reconciliation is the process React uses to compare the old Virtual DOM with the new Virtual DOM.

Its goal is to determine:

- What changed?
- What should be updated?
- What should remain the same?

React updates only necessary DOM elements.

---

# 17. What is the Diffing Algorithm?

## Answer

Diffing Algorithm is the comparison algorithm used during reconciliation.

Instead of comparing every node deeply, React uses optimized assumptions:

### Rule 1

Different element types create different trees.

```jsx
<div>

↓

<span>
```

Entire subtree recreated.

---

### Rule 2

Same element type

Only changed attributes are updated.

```jsx
<button disabled />

↓

<button />
```

Only disabled attribute changes.

---

### Rule 3

Keys help React identify list items.

```jsx
users.map(user =>
    <User key={user.id} />
)
```

Keys improve performance.

---

# 18. How does React identify which DOM node changed?

## Answer

React identifies changed nodes using:

- Element Type
- Props
- State
- Key

Example

```jsx
<li key={1}>Apple</li>

<li key={2}>Banana</li>
```

If Banana changes,

React only updates that node.

Without keys,

React may unnecessarily recreate list items.

---

# 19. What are React Elements?

## Answer

React Element is a plain JavaScript object that describes what should appear on the screen.

JSX

```jsx
<h1>Hello</h1>
```

is converted into

```javascript
React.createElement(
    "h1",
    null,
    "Hello"
);
```

which produces

```javascript
{
  type: "h1",
  props: {
      children: "Hello"
  }
}
```

React Elements are immutable.

---

# 20. Difference between React Element and DOM Element

| React Element | DOM Element |
|---------------|-------------|
| JavaScript Object | Actual HTML Node |
| Lightweight | Heavy |
| Immutable | Mutable |
| Exists in memory | Exists in browser |
| Created by React | Created by browser |

---

# Interview Tip

A common interview question is:

**Q: Explain how React updates the UI internally.**

### Answer

1. User performs an action.
2. State or props change.
3. Component re-renders.
4. React creates a new Virtual DOM.
5. React compares it with the old Virtual DOM using the Diffing Algorithm.
6. Reconciliation identifies the changed nodes.
7. React updates only those nodes in the Real DOM.
8. Browser repaints the updated UI.

This process makes React fast and efficient.