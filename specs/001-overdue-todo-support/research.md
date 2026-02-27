# Research: Support for Overdue Todo Items

**Feature**: 001-overdue-todo-support  
**Phase**: 0 — Outline & Research  
**Date**: 2026-02-27

---

## 1. Overdue Date Comparison in JavaScript

**Question**: How should we compare a todo's due date string against today's local date reliably?

**Decision**: Compare date strings in `YYYY-MM-DD` format without converting to full timestamps.

**Rationale**:
- The todo `dueDate` field is already stored as a `YYYY-MM-DD` ISO date string (confirmed in `todoService.js`).
- `new Date().toLocaleDateString('sv')` (Swedish locale) reliably returns a `YYYY-MM-DD` string using the user's locale — no timezone conversion needed.
- String comparison of `YYYY-MM-DD` values (`dueDate < todayStr`) is lexicographically correct and avoids `Date` parsing pitfalls (midnight UTC vs. local midnight).
- The spec explicitly states: "comparison uses the user's local date as reported by their browser/device."

**Alternatives considered**:
- `new Date(dueDate) < new Date()` — rejected: `new Date('2026-02-26')` is parsed as UTC midnight, which may evaluate as "not yet past" if the user is UTC+N before 00:00 UTC.
- `Date.now()` comparison — same timezone issue.
- Moment.js / date-fns — rejected (Principle V: YAGNI; adds a dependency for three lines of logic).

**Implementation**:
```js
// utils/overdueUtils.js
export function getTodayString() {
  return new Date().toLocaleDateString('sv'); // 'sv' locale → 'YYYY-MM-DD'
}

export function isOverdue(todo, todayStr) {
  if (!todo.dueDate) return false;
  if (todo.completed) return false;
  return todo.dueDate < todayStr;
}
```

---

## 2. WCAG 1.4.1 Compliance for Overdue Indicators

**Question**: What dual-signal pattern satisfies WCAG 1.4.1 without adding unnecessary complexity?

**Decision**: Render a small inline `<span className="overdue-badge">Overdue</span>` inside the `TodoCard`, styled with a distinct background color token AND the text label "Overdue".

**Rationale**:
- WCAG 1.4.1 (Use of Color) requires that color is not the sole means of conveying information. A visible text label "Overdue" makes the state unambiguous for color-blind users and screen readers.
- Using a CSS class on the card (e.g., `todo-card--overdue`) to change background/border color, combined with the badge text, delivers both signals.
- An `aria-label` or `role="status"` on the badge is not strictly required by WCAG 1.4.1 but will be added for completeness (screen readers announce badge text naturally).

**Alternatives considered**:
- Icon (⚠) only — rejected (icon alone may be insufficient for screen readers without alt text; text label is simpler and unambiguous).
- Color border only — rejected (violates WCAG 1.4.1: color alone).

---

## 3. Overdue Count Banner: Placement and Visibility

**Question**: How should the overdue count banner be implemented inside `TodoList`?

**Decision**: Conditionally render a `<div className="overdue-banner">` above the todo cards in `TodoList`. Calculate count inline from the `todos` prop and `isOverdue()`.

**Rationale**:
- The spec directs: "Top of the todo list (inline summary row or banner above the list items)" and "Hide the summary row/banner entirely when overdue count is zero."
- Computing the count in `TodoList` from `todos` keeps it in sync without extra state or effects (count is purely derived from props already available).
- No prop drilling needed: `TodoList` already receives the full `todos` array.

**Alternatives considered**:
- Compute count in `App.js` and pass as prop — rejected (minor prop drilling, no benefit; `TodoList` already has everything it needs).
- Store overdue count in backend — rejected (spec says "derived — not stored"; Principle V).

---

## 4. When to Compute Overdue Status

**Question**: Should `getTodayString()` be called once at module load, once per render, or via a hook?

**Decision**: Call `getTodayString()` once per render cycle (inside the component render, not in a `useEffect` or module-level constant).

**Rationale**:
- The spec says "once on page load." In React, this effectively means once when the component tree first mounts. Calling it on each render has negligible cost and ensures the value is freshly captured for every render.
- A module-level constant would be stale if tests run across midnight (unlikely but fragile).
- A `useEffect` with no deps and `useState` would be over-engineering for this use case (Principle V: KISS).

---

## 5. Test Strategy Alignment

**Question**: How do we test `isOverdue()` deterministically without relying on the real system clock?

**Decision**: Pass `todayStr` as an explicit parameter to `isOverdue(todo, todayStr)` and inject a fixed date string in tests.

**Rationale**:
- Avoids mocking `Date` globally (fragile, affects other tests in the suite).
- Pure function with injected dependency is easier to test and reason about.
- In components, `getTodayString()` is called at the call site and can be mocked via module mocking or by extracting it into the component's render scope.

**Pattern**:
```js
// In tests:
expect(isOverdue({ dueDate: '2026-02-26', completed: false }, '2026-02-27')).toBe(true);
expect(isOverdue({ dueDate: '2026-02-27', completed: false }, '2026-02-27')).toBe(false); // today is NOT overdue
expect(isOverdue({ dueDate: '2026-02-26', completed: true  }, '2026-02-27')).toBe(false); // completed excluded
expect(isOverdue({ dueDate: null,         completed: false }, '2026-02-27')).toBe(false); // no due date
```

---

## 6. CSS Token Strategy for Overdue Styling

**Question**: Which theme.css tokens should be used or added for overdue colors?

**Decision**: Add two new semantic tokens to `theme.css`: `--color-overdue-bg` and `--color-overdue-text`, defined for both light and dark modes.

**Rationale**:
- Constitution Principle IV: "all colors MUST reference the defined palette variables; no raw hex values in component CSS unless defining the token itself."
- Red/amber tones are conventional for overdue indicators and meet WCAG AA contrast (4.5:1) with white text on a sufficiently dark background.
- Defining tokens in `theme.css` means dark-mode support is handled automatically.

**Proposed token values**:
```css
/* light mode */
--color-overdue-bg:   #fde8e8;
--color-overdue-text: #b91c1c;  /* red-700, contrast ratio ≥ 4.5:1 on white bg */

/* dark mode */
--color-overdue-bg:   #450a0a;
--color-overdue-text: #fca5a5;  /* red-300, readable on dark bg */
```

---

## Summary of Decisions

| Decision | Choice |
|----------|--------|
| Date comparison method | `YYYY-MM-DD` string compare via `toLocaleDateString('sv')` |
| Overdue signal | Color class on card + "Overdue" text badge |
| Banner location | Top of `TodoList`, hidden when count = 0 |
| Today computation | Per-render call to `getTodayString()` |
| Test isolation | `todayStr` injected as parameter to `isOverdue()` |
| CSS tokens | Two new semantic tokens in `theme.css` |
| Backend changes | None |
