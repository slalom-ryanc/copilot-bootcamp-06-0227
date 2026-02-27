# Quickstart: Implementing Overdue Todo Support

**Feature**: 001-overdue-todo-support  
**Phase**: 1 — Design  
**Date**: 2026-02-27

This guide gives implementers a fast ramp-up on the implementation order, key decisions, and verification steps for this feature. For full details see [research.md](research.md), [data-model.md](data-model.md), and [contracts/overdue-ui-contract.md](contracts/overdue-ui-contract.md).

---

## Prerequisites

```bash
# Ensure you are on the correct branch
git checkout 001-overdue-todo-support

# Install dependencies (if not already done)
npm install

# Run existing tests to confirm green baseline
npm test
```

---

## Implementation Order

Follow this order to stay test-first and incremental:

### Step 1 — Create `overdueUtils` with tests first

**File**: `packages/frontend/src/utils/overdueUtils.js`  
**Test**: `packages/frontend/src/utils/__tests__/overdueUtils.test.js`

Write the test file first. Cover all five behavioral contracts from the UI contract:

```js
// overdueUtils.test.js
import { isOverdue } from '../overdueUtils';

describe('isOverdue', () => {
  const TODAY = '2026-02-27';

  it('returns true for incomplete todo with past due date', () => {
    expect(isOverdue({ dueDate: '2026-02-26', completed: false }, TODAY)).toBe(true);
  });
  it('returns false for incomplete todo due today', () => {
    expect(isOverdue({ dueDate: TODAY, completed: false }, TODAY)).toBe(false);
  });
  it('returns false for incomplete todo with future due date', () => {
    expect(isOverdue({ dueDate: '2026-03-01', completed: false }, TODAY)).toBe(false);
  });
  it('returns false for todo with no due date', () => {
    expect(isOverdue({ dueDate: null, completed: false }, TODAY)).toBe(false);
  });
  it('returns false for completed todo with past due date', () => {
    expect(isOverdue({ dueDate: '2026-02-26', completed: true }, TODAY)).toBe(false);
  });
});
```

Then implement `overdueUtils.js`:

```js
/**
 * Returns the user's local current date as a YYYY-MM-DD string.
 * Uses Swedish locale ('sv') which formats dates as YYYY-MM-DD.
 * @returns {string} Today's date in YYYY-MM-DD format
 */
export function getTodayString() {
  return new Date().toLocaleDateString('sv');
}

/**
 * Determines whether a todo item is overdue.
 * A todo is overdue when it is incomplete and its due date is strictly before today.
 * @param {{ dueDate: string|null, completed: boolean }} todo
 * @param {string} todayStr - Today's date in YYYY-MM-DD format
 * @returns {boolean}
 */
export function isOverdue(todo, todayStr) {
  if (!todo.dueDate) return false;
  if (todo.completed) return false;
  return todo.dueDate < todayStr;
}
```

### Step 2 — Add CSS tokens to `theme.css`

**File**: `packages/frontend/src/styles/theme.css`

Add to `:root` (light) and `[data-theme="dark"]` selectors:

```css
/* Overdue indicator tokens */
--color-overdue-bg:   #fde8e8;
--color-overdue-text: #b91c1c;
```

```css
/* Inside [data-theme="dark"] */
--color-overdue-bg:   #450a0a;
--color-overdue-text: #fca5a5;
```

Add component classes (e.g., at end of theme.css or in a dedicated section):

```css
.todo-card--overdue {
  border-left: 4px solid var(--color-overdue-text);
  background-color: var(--color-overdue-bg);
}

.overdue-badge {
  display: inline-block;
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--color-overdue-text);
  background-color: var(--color-overdue-bg);
  border: 1px solid var(--color-overdue-text);
  border-radius: 4px;
  padding: 2px 6px;
  margin-left: 8px;
}

.overdue-banner {
  background-color: var(--color-overdue-bg);
  color: var(--color-overdue-text);
  border: 1px solid var(--color-overdue-text);
  border-radius: 4px;
  padding: 8px 16px;
  margin-bottom: 16px;
  font-weight: 600;
}
```

### Step 3 — Update `TodoCard` (extend tests first)

**File**: `packages/frontend/src/components/TodoCard.js`  
**Test**: `packages/frontend/src/components/__tests__/TodoCard.test.js`

Add `overdueUtils` import, compute `overdue` from props, apply CSS class and badge:

Key changes:
1. `import { isOverdue, getTodayString } from '../utils/overdueUtils';`
2. Inside the component (view mode): `const overdue = isOverdue(todo, getTodayString());`
3. Apply `todo-card--overdue` class: `className={\`todo-card ${overdue ? 'todo-card--overdue' : ''}\`}`
4. Render badge: `{overdue && <span className="overdue-badge">Overdue</span>}`

In tests, mock `getTodayString` to return a fixed date:
```js
jest.mock('../utils/overdueUtils', () => ({
  ...jest.requireActual('../utils/overdueUtils'),
  getTodayString: () => '2026-02-27',
}));
```

### Step 4 — Update `TodoList` (extend tests first)

**File**: `packages/frontend/src/components/TodoList.js`  
**Test**: `packages/frontend/src/components/__tests__/TodoList.test.js`

1. Import utils: `import { isOverdue, getTodayString } from '../utils/overdueUtils';`
2. Compute count at top of render: `const todayStr = getTodayString(); const overdueCount = todos.filter(t => isOverdue(t, todayStr)).length;`
3. Render banner before todo cards:
   ```jsx
   {overdueCount > 0 && (
     <div className="overdue-banner">{overdueCount} overdue</div>
   )}
   ```

---

## Verification Checklist

```bash
# 1. Lint — zero errors required
npm run lint

# 2. Tests — all pass, ≥ 80% coverage
npm test -- --coverage

# 3. Manual smoke test
#    - Start the app: npm start (from repo root or packages/frontend)
#    - Create a todo with yesterday's date → should show "Overdue" badge and red styling
#    - Create a todo with today's date → should NOT show overdue
#    - Create a todo with no due date → should NOT show overdue
#    - Mark the overdue todo complete → overdue indicator disappears immediately
#    - Edit overdue todo's due date to tomorrow → overdue indicator disappears
#    - Verify overdue count banner shows correct number
#    - When no overdue todos → banner is hidden
```

---

## Key Files Summary

| File | Action | Notes |
|------|--------|-------|
| `packages/frontend/src/utils/overdueUtils.js` | CREATE | Pure utility; `isOverdue()` + `getTodayString()` |
| `packages/frontend/src/utils/__tests__/overdueUtils.test.js` | CREATE | 5 unit tests minimum |
| `packages/frontend/src/styles/theme.css` | EDIT | Add `--color-overdue-*` tokens and `.overdue-badge`, `.todo-card--overdue`, `.overdue-banner` classes |
| `packages/frontend/src/components/TodoCard.js` | EDIT | Add overdue class + badge (view mode only, not edit mode) |
| `packages/frontend/src/components/__tests__/TodoCard.test.js` | EDIT | Add overdue/non-overdue scenarios |
| `packages/frontend/src/components/TodoList.js` | EDIT | Add overdue count banner |
| `packages/frontend/src/components/__tests__/TodoList.test.js` | EDIT | Add banner show/hide scenarios |
| `packages/backend/` | NO CHANGE | Feature is frontend-only |
