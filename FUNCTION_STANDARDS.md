# Function Standards & Patterns

Engineering reference for consistent code practices in this codebase.

---

## Table of Contents

1. [Naming Conventions](#naming-conventions)
2. [Function Signatures](#function-signatures)
3. [Error Handling](#error-handling)
4. [State Management](#state-management)
5. [Component Organization](#component-organization)
6. [Import & Export Patterns](#import--export-patterns)
7. [Testing Standards](#testing-standards)
8. [Comments & Documentation](#comments--documentation)
9. [Constants](#constants)
10. [Styling Conventions](#styling-conventions)

---

## Naming Conventions

### Functions — `camelCase`

All functions use camelCase. Name functions to clearly describe their action.

```js
// Event handlers: prefix with "handle"
const handleInputChange = (field, value) => { ... };
const handleSubmit = () => { ... };

// Action functions: verb + noun
const addSpending = () => { ... };
const toggleCurrency = () => { ... };
const deleteSpending = (id) => { ... };

// Getter/compute functions: prefix with "get"
const getTotalByCategory = (category) => { ... };

// Formatters/utilities: named for their purpose
const formatAmount = (amount, currency) => { ... };
```

### React State — `camelCase` with `set` prefix for setters

```js
const [spendings, setSpendings] = useState([]);
const [filterCategory, setFilterCategory] = useState('All');
const [formData, setFormData] = useState({});
```

### Constants — `UPPER_SNAKE_CASE`

```js
const CATEGORIES = ['General', 'Personal', 'Auto', 'House'];
const CURRENCIES = ['MXN', 'USD'];
```

### Variables — `camelCase`

```js
const filteredSpendings = spendings.filter(...);
const isFormValid = formData.amount && parseFloat(formData.amount) > 0;
```

---

## Function Signatures

### Arrow Functions

Prefer arrow functions for handlers and callbacks defined inside components.

```js
// No parameters
const toggleCurrency = () => { ... };

// Single parameter (no parens needed, but parens are acceptable too)
const deleteSpending = (id) => { ... };

// Multiple parameters
const handleInputChange = (field, value) => { ... };
```

### Named Function Declarations

Use named function declarations for top-level React components.

```js
function App() {
  // state, handlers, computed values...
  return <JSX />;
}

export default App;
```

### No TypeScript

This project uses plain JavaScript. Do not add TypeScript type annotations. Parameter names should be descriptive enough to communicate intent.

```js
// Correct — descriptive names without types
const formatAmount = (amount, currency) => { ... };

// Avoid — unnecessary type casting
const formatAmount = (amount: number, currency: string): string => { ... };
```

### No Async/Await (unless adding network calls)

Current code has no asynchronous operations. If you introduce network requests or async logic, use `async/await` consistently — do not mix with raw Promise chains.

```js
// If adding async operations:
const fetchSpendings = async () => {
  const data = await api.getSpendings();
  setSpendings(data);
};
```

---

## Error Handling

### Principle: Prevent Over Catch

Validate inputs before they reach logic — don't rely on catching thrown errors.

### 1. Guard Clauses — Early Return

Use early returns to reject invalid states at the top of a function.

```js
const addSpending = () => {
  if (!formData.amount || parseFloat(formData.amount) <= 0) {
    alert('Please enter a valid amount');
    return;  // Exit early; don't proceed
  }
  // Happy path continues here
};
```

### 2. Input Sanitization

Strip invalid characters as the user types — prevent bad state from entering the system.

```js
const handleInputChange = (field, value) => {
  if (field === 'amount') {
    // Only allow digits and a single decimal point
    const numericValue = value.replace(/[^0-9.]/g, '');
    const parts = numericValue.split('.');
    if (parts.length > 2) {
      return;  // Reject multiple decimal points
    }
    // ...
  }
};
```

### 3. UI-Level Prevention via Disabled State

Disable controls when preconditions are not met — don't let users submit invalid forms.

```js
const isFormValid = formData.amount && parseFloat(formData.amount) > 0;

<button disabled={!isFormValid} onClick={addSpending}>
  Add
</button>
```

### 4. User Feedback via `alert()`

Use the browser's `alert()` for simple validation messages. More complex UI feedback (toast, inline errors) can be introduced later.

```js
alert('Please enter a valid amount');
```

> In tests, `alert` is mocked globally via `vi.fn()` in `src/test/setup.js`.

---

## State Management

Use React `useState` exclusively. No external state libraries.

### Initialization

Define all state variables at the top of the component, before handlers.

```js
function App() {
  const [spendings, setSpendings] = useState([]);
  const [filterCategory, setFilterCategory] = useState('All');
  const [currency, setCurrency] = useState('MXN');
  const [formData, setFormData] = useState({
    amount: '',
    category: 'General',
    description: '',
  });
  // ...
}
```

### Immutable Updates

Never mutate state directly. Use spread or array methods to produce new values.

```js
// Adding an item
setSpendings([...spendings, newSpending]);

// Removing an item
setSpendings(spendings.filter(s => s.id !== id));

// Updating a field in an object
setFormData({ ...formData, [field]: value });
```

### Computed Values

Derive values from state inline — don't store derived data in separate state variables.

```js
// Derived from state, not a separate useState
const filteredSpendings = filterCategory === 'All'
  ? spendings
  : spendings.filter(spending => spending.category === filterCategory);

const total = filteredSpendings.reduce((sum, s) => sum + s.amount, 0);
```

---

## Component Organization

All code lives in a single component file (`App.jsx`). Follow this internal ordering:

```
1. Imports
2. Module-level constants (CATEGORIES, CURRENCIES)
3. Component function declaration
   a. State (useState)
   b. Event handlers / action functions
   c. Computed/derived values
   d. Return statement (JSX)
5. Default export
```

Example skeleton:

```js
import React, { useState } from 'react';

const CATEGORIES = ['General', 'Personal', 'Auto', 'House'];
const CURRENCIES = ['MXN', 'USD'];

function App() {
  // a. State
  const [spendings, setSpendings] = useState([]);

  // b. Handlers
  const handleInputChange = (field, value) => { ... };
  const addSpending = () => { ... };

  // c. Derived values
  const filteredSpendings = spendings.filter(...);

  // d. Render
  return (
    <div className="container">
      ...
    </div>
  );
}

export default App;
```

### Conditional Rendering

Use ternaries for simple if/else rendering. Use `&&` for optional sections.

```js
// Ternary: empty state vs. list
{filteredSpendings.length === 0 ? (
  <div className="empty-state">No spendings yet</div>
) : (
  filteredSpendings.map(spending => (
    <div key={spending.id}>...</div>
  ))
)}

// &&: render only when condition is true
{isFormValid && <span className="hint">Ready to add</span>}
```

---

## Import & Export Patterns

### Imports

Group in this order, separated by a blank line where helpful:
1. React and React hooks
2. Third-party libraries
3. Local modules

```js
// 1. React
import React, { useState } from 'react';

// 2. Third-party (in tests)
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// 3. Local
import App from './App';
```

### Exports

Use default exports for components. No named exports from component files.

```js
// Correct
export default App;

// Avoid in component files
export { App };
export const helper = () => {};
```

---

## Testing Standards

Tests use **Vitest** + **React Testing Library**. Follow the AAA (Arrange–Act–Assert) pattern.

### File Naming

| File | Purpose |
|------|---------|
| `App.test.jsx` | Passing tests for implemented features |
| `App.failing.test.jsx` | Specs for planned features (intentionally failing) |
| `EdgeCases.failing.test.jsx` | Advanced/edge case specs (intentionally failing) |

Failing test files act as living specifications — implement the feature, move/refactor the tests to the passing file.

### Test Structure

```js
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, beforeEach, vi } from 'vitest';
import App from './App';

describe('Feature Group', () => {
  beforeEach(() => {
    vi.clearAllMocks();  // Always reset mocks between tests
  });

  describe('Sub-group', () => {
    it('describes the expected behavior', async () => {
      // Arrange
      const user = userEvent.setup();
      render(<App />);

      // Act
      await user.type(screen.getByTestId('amount-input'), '50');
      await user.click(screen.getByTestId('add-spending-btn'));

      // Assert
      expect(screen.getByText('$50.00 MXN')).toBeInTheDocument();
    });
  });
});
```

### Query Priority

Use these queries in order of preference:

1. `getByLabelText()` — form inputs with labels
2. `getByText()` — visible text content
3. `getByRole()` — semantic HTML roles
4. `getByPlaceholderText()` — inputs by placeholder
5. `getByTestId()` — last resort, or for complex selections

### `userEvent` Over `fireEvent`

Prefer `userEvent` for realistic user interactions. It correctly simulates focus, keystrokes, and browser events. Use `fireEvent` only when `userEvent` is insufficient.

```js
// Preferred
const user = userEvent.setup();
await user.type(input, 'hello');
await user.click(button);

// Avoid unless necessary
fireEvent.change(input, { target: { value: 'hello' } });
```

### `data-testid` Attributes

Add `data-testid` to all interactive elements and key UI containers so tests can reliably select them.

```jsx
<input data-testid="amount-input" ... />
<select data-testid="category-select" ... />
<button data-testid="add-spending-btn" ... />
```

### Mocking

Mock `alert` globally in `src/test/setup.js` — do not mock it in individual tests.

```js
// src/test/setup.js
import '@testing-library/jest-dom';
global.alert = vi.fn();
```

---

## Comments & Documentation

### Inline Comments — Sparingly

Only comment on non-obvious logic. Self-explanatory code needs no comment.

```js
// Correct: explains the "why"
// Only allow digits and a single decimal point
const numericValue = value.replace(/[^0-9.]/g, '');

// Avoid: restates what the code already says
// Filter spendings by category
const filtered = spendings.filter(s => s.category === filterCategory);
```

### JSX Section Comments

Use JSX comments `{/* */}` to label major sections of a large render block.

```jsx
return (
  <div className="container">
    {/* Header */}
    <h1>Spending Tracker</h1>

    {/* Add Spending Form */}
    <div className="card">
      ...
    </div>

    {/* Spending List */}
    <div className="card">
      ...
    </div>
  </div>
);
```

### No JSDoc

This project does not use JSDoc. Keep function names and parameter names descriptive enough to be self-documenting.

---

## Constants

Define module-level constants (outside the component) when values are static and reused.

```js
// Outside component — computed once, not on every render
const CATEGORIES = ['General', 'Personal', 'Auto', 'House'];
const CURRENCIES = ['MXN', 'USD'];

function App() { ... }
```

Do not inline magic values in JSX or logic if they are used in more than one place.

```js
// Avoid
if (currency === 'USD') { ... }  // Magic string repeated

// Prefer
const DEFAULT_CURRENCY = 'USD';
if (currency === DEFAULT_CURRENCY) { ... }
```

---

## Styling Conventions

Styles are defined in `index.html` as a `<style>` block. There are no CSS modules, Tailwind classes, or separate `.css` files.

### Class Names — kebab-case

```jsx
<div className="container">
<div className="card">
<span className="empty-state">
<button className="btn-primary">
```

### Class Naming — Descriptive, BEM-Inspired

Use descriptive class names that reflect the component's role or state. Modifier classes can be appended.

```jsx
<button className="btn btn-primary" disabled={!isFormValid}>
<div className="spending-item spending-item--highlighted">
```

### No Inline Styles

Avoid inline `style={{}}` props unless the value is truly dynamic (e.g., a user-defined color). Use CSS classes for all static visual styling.

```jsx
// Avoid
<div style={{ marginTop: '16px', color: '#333' }}>

// Prefer — define in index.html <style> block
<div className="section-header">
```

---

## Quick Reference

| Topic | Pattern |
|-------|---------|
| Function names | `camelCase` |
| Event handlers | Prefix with `handle` — `handleInputChange` |
| Action functions | Verb + noun — `addSpending`, `toggleCurrency` |
| Getters | Prefix with `get` — `getTotalByCategory` |
| Constants | `UPPER_SNAKE_CASE` |
| State setters | Prefix with `set` — `setSpendings` |
| Component type | Functional (no class components) |
| Exports | Default export only |
| Error strategy | Prevent via validation; guard clauses; `alert()` |
| State mutations | Immutable — spread/filter, never mutate directly |
| Test pattern | AAA; `userEvent`; `getByLabelText` > `getByTestId` |
| Comments | Only for non-obvious logic |
| Styling | Classes in `index.html`; kebab-case names |
