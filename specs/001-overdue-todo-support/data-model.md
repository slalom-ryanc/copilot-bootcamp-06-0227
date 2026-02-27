# Data Model: Support for Overdue Todo Items

**Feature**: 001-overdue-todo-support  
**Phase**: 1 — Design  
**Date**: 2026-02-27

---

## Existing Entity: Todo Item

No changes to the stored data model are required. The `Todo` object is already defined in the backend `todoService.js` and surfaced to the frontend via the REST API.

### Current Todo Shape (frontend-visible)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | `number` | ✅ | Auto-incrementing integer identifier |
| `title` | `string` | ✅ | Todo text; 1–255 characters |
| `completed` | `boolean` | ✅ | `true` if the todo has been marked done |
| `dueDate` | `string \| null` | ❌ | ISO date string `YYYY-MM-DD`; `null` if no due date set |
| `createdAt` | `string` | ✅ | ISO timestamp of creation |

---

## Derived Property: Overdue Status

Overdue status is **not stored** — it is computed at render time from the existing `Todo` fields.

### Definition

```
isOverdue(todo) = true  iff
  todo.dueDate !== null
  AND todo.completed === false
  AND todo.dueDate < todayString   // strict: today is NOT overdue
```

where `todayString` is the user's local date in `YYYY-MM-DD` format, captured once per render using `new Date().toLocaleDateString('sv')`.

### Truth Table

| `completed` | `dueDate` | `dueDate < today` | `isOverdue` |
|-------------|-----------|-------------------|-------------|
| `false` | `"2026-02-26"` | ✅ (yesterday) | **true** |
| `false` | `"2026-02-27"` | ❌ (today) | false |
| `false` | `"2026-03-01"` | ❌ (future) | false |
| `false` | `null` | N/A | false |
| `true` | `"2026-02-26"` | ✅ | false (completed excludes overdue) |
| `true` | `null` | N/A | false |

---

## New Module: `overdueUtils`

**Location**: `packages/frontend/src/utils/overdueUtils.js`

### Exported Functions

#### `getTodayString(): string`

Returns the current local date as a `YYYY-MM-DD` string. No parameters.

| Concern | Detail |
|---------|--------|
| Side effects | None (reads `Date`) |
| Timezone | Uses browser local date via Swedish locale trick |
| Usage | Called at render time, result passed to `isOverdue()` |

#### `isOverdue(todo: TodoObject, todayStr: string): boolean`

Pure function. Returns `true` if the todo is overdue relative to `todayStr`.

| Parameter | Type | Description |
|-----------|------|-------------|
| `todo` | `Object` | Todo object with `dueDate` and `completed` fields |
| `todayStr` | `string` | Local today date in `YYYY-MM-DD` format |

| Return | Condition |
|--------|-----------|
| `false` | `todo.dueDate` is null/undefined/falsy |
| `false` | `todo.completed` is truthy |
| `false` | `todo.dueDate >= todayStr` |
| `true` | `todo.dueDate < todayStr` AND `!todo.completed` |

---

## State Transitions: Overdue Status Lifecycle

```
Todo created (no due date) ──────────────────────────────→ never overdue

Todo created (future due date) ──── page reload after expiry ──→ isOverdue = true
                                        │
                                  user completes todo ───────→ isOverdue = false
                                        │
                                  user edits due date to future ─→ isOverdue = false
                                        │
                                  user deletes todo ──────────→ removed from list
```

All transitions are reflected immediately in the next React render (no async update needed — overdue is re-derived from the mutated `todos` array on every render).

---

## Overdue Count (Derived Aggregate)

| Property | Value |
|----------|-------|
| `overdueCount` | `todos.filter(t => isOverdue(t, todayStr)).length` |
| Computed in | `TodoList` component render body |
| Stored | ❌ Not stored — recalculated each render |
| Display condition | Rendered only when `overdueCount > 0` |
