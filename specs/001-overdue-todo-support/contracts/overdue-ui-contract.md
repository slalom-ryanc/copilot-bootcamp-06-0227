# UI Contract: Overdue Todo Indicators

**Feature**: 001-overdue-todo-support  
**Phase**: 1 — Design  
**Date**: 2026-02-27

This document defines the observable UI contracts (component props, CSS classes, DOM structure, and accessibility attributes) that implementation must satisfy. Tests should assert these contracts.

---

## Component: `TodoCard`

### Props (unchanged)

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `todo` | `Object` | ✅ | Todo data including `dueDate` and `completed` |
| `onToggle` | `Function` | ✅ | Called with `todo.id` when checkbox toggled |
| `onEdit` | `Function` | ✅ | Called with `(id, title, dueDate)` on edit submit |
| `onDelete` | `Function` | ✅ | Called with `todo.id` on delete |
| `isLoading` | `boolean` | ❌ | Disables controls when true |

### Visual Overdue State

When `isOverdue(todo, getTodayString())` returns `true`, the rendered card **MUST**:

1. Have the CSS class `todo-card--overdue` on the root element (alongside `todo-card`).
2. Contain a `<span className="overdue-badge">` with the text `Overdue` as its text content.
3. The badge MUST be visible (not hidden) and appear alongside or below the todo title.

When `isOverdue` returns `false`, the rendered card **MUST NOT**:
- Include the class `todo-card--overdue`.
- Render any element with `className="overdue-badge"`.

### DOM Structure (overdue state)

```html
<div class="todo-card todo-card--overdue">
  <div class="todo-content">
    <input type="checkbox" aria-label="Mark as complete" />
    <div class="todo-details">
      <span class="todo-title">Buy groceries</span>
      <span class="overdue-badge">Overdue</span>      <!-- NEW -->
      <span class="todo-due-date">February 26, 2026</span>
    </div>
  </div>
  ...
</div>
```

### Accessibility Requirements

- `overdue-badge` text must be accessible to screen readers (no `aria-hidden`).
- Color contrast of badge text on badge background MUST meet WCAG AA (≥ 4.5:1).
- No additional ARIA roles required (text content alone satisfies 1.4.1).

---

## Component: `TodoList`

### Props (unchanged)

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `todos` | `Array` | ✅ | Array of todo objects |
| `onToggle` | `Function` | ✅ | Passed through to each `TodoCard` |
| `onEdit` | `Function` | ✅ | Passed through to each `TodoCard` |
| `onDelete` | `Function` | ✅ | Passed through to each `TodoCard` |
| `isLoading` | `boolean` | ❌ | Passed through to each `TodoCard` |

### Overdue Count Banner

When `overdueCount > 0`, the rendered list **MUST**:

1. Include a `<div className="overdue-banner">` as the **first child** of the list container (before any todo cards).
2. The banner text **MUST** match the pattern `{n} overdue` (e.g., `"3 overdue"`, `"1 overdue"`).

When `overdueCount === 0`, the rendered list **MUST NOT**:
- Render any element with `className="overdue-banner"`.
- Display any text containing "0 overdue".

### DOM Structure (with overdue banner)

```html
<div class="todo-list">
  <div class="overdue-banner">2 overdue</div>   <!-- NEW: only when count > 0 -->
  <div class="todo-card todo-card--overdue">...</div>
  <div class="todo-card">...</div>
  <div class="todo-card todo-card--overdue">...</div>
</div>
```

### Accessibility Requirements

- Banner text is plain text content — no `aria-live` required (status is set on page load).
- Banner MUST have sufficient color contrast (WCAG AA).

---

## Utility Module: `overdueUtils`

### Exported API Contract

```typescript
// Conceptual TypeScript signature (implemented in plain JS)

/**
 * Returns the user's local current date as a YYYY-MM-DD string.
 * Must use the browser's local timezone.
 */
function getTodayString(): string;

/**
 * Returns true if the given todo is overdue relative to todayStr.
 * A todo is overdue iff: dueDate is set, dueDate < todayStr, and todo is not completed.
 * "today" (dueDate === todayStr) is NOT overdue.
 */
function isOverdue(todo: { dueDate: string | null, completed: boolean }, todayStr: string): boolean;
```

### Behavioral Contracts (testable assertions)

```
isOverdue({ dueDate: 'YESTERDAY', completed: false }, 'TODAY') === true
isOverdue({ dueDate: 'TODAY',     completed: false }, 'TODAY') === false
isOverdue({ dueDate: 'TOMORROW',  completed: false }, 'TODAY') === false
isOverdue({ dueDate: null,        completed: false }, 'TODAY') === false
isOverdue({ dueDate: 'YESTERDAY', completed: true  }, 'TODAY') === false
```

---

## CSS Tokens (theme.css additions)

| Token | Light Value | Dark Value | Usage |
|-------|------------|------------|-------|
| `--color-overdue-bg` | `#fde8e8` | `#450a0a` | Background of `.overdue-badge` and `.todo-card--overdue` accent |
| `--color-overdue-text` | `#b91c1c` | `#fca5a5` | Text color of `.overdue-badge`; border/accent of overdue card |

These tokens MUST be defined in `theme.css` within the `:root` (light) and `[data-theme="dark"]` (dark) selectors.

---

## Post-Design Constitution Re-Check

| # | Principle | Status |
|---|-----------|--------|
| I | Clean Code & Consistent Style | ✅ JSDoc on exported utils; 2-space indent; naming conventions respected |
| II | Test-First with 80%+ Coverage | ✅ `overdueUtils.test.js`, extended `TodoCard.test.js`, extended `TodoList.test.js` required before merge |
| III | Single Responsibility | ✅ `isOverdue` is pure logic; `TodoCard` is presentation; no component fetches data |
| IV | Accessible & Themed UI | ✅ WCAG 1.4.1: dual signal (color class + badge text); `--color-overdue-*` tokens defined for light/dark |
| V | Scope Discipline | ✅ No backend changes; no new endpoints; overdue is derived, not stored |

**Post-design gate result: ALL PASS.**
