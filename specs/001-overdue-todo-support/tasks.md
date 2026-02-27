# Tasks: Support for Overdue Todo Items

**Input**: Design documents from `/specs/001-overdue-todo-support/`
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [data-model.md](data-model.md), [contracts/overdue-ui-contract.md](contracts/overdue-ui-contract.md), [research.md](research.md), [quickstart.md](quickstart.md)

**Scope**: Frontend-only feature (no backend changes). All overdue logic is derived at render time using a pure utility function.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies on incomplete tasks)
- **[Story]**: Which user story this task belongs to (US1, US2)
- Exact file paths included in all descriptions

---

## Phase 1: Setup

**Purpose**: Verify the existing baseline is green before introducing any changes.

- [ ] T001 Create feature branch `001-overdue-todo-support` and confirm existing tests pass (`npm test` from repo root)

**Checkpoint**: All pre-existing tests are green — safe to begin development.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: The `overdueUtils` module and CSS design tokens are shared infrastructure consumed by **both** User Story 1 and User Story 2. Neither story can be implemented until this phase is complete.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T002 Write unit tests for `overdueUtils` in `packages/frontend/src/utils/__tests__/overdueUtils.test.js` (tests must FAIL before T003 is implemented — cover all 5 behavioral contracts from the UI contract)
- [ ] T003 Implement `packages/frontend/src/utils/overdueUtils.js` — export `getTodayString()` and `isOverdue(todo, todayStr)` as pure functions per the data-model spec
- [ ] T004 [P] Add overdue CSS design tokens (`--color-overdue-bg`, `--color-overdue-text`) and component classes (`.todo-card--overdue`, `.overdue-badge`, `.overdue-banner`) to `packages/frontend/src/styles/theme.css` for both `:root` (light) and `[data-theme="dark"]` selectors

**Checkpoint**: `overdueUtils` tests pass, CSS tokens defined — user story implementation can now begin in parallel.

---

## Phase 3: User Story 1 — Visual Overdue Indicators (Priority: P1) 🎯 MVP

**Goal**: Incomplete todos whose due date is strictly before today are visually distinguished from all other todos using both a color change (`todo-card--overdue` CSS class) and a text label (`<span className="overdue-badge">Overdue</span>`), satisfying WCAG 1.4.1.

**Independent Test**: Create a todo with a due date in the past and a todo with a future due date. View the list and confirm the past-due todo shows the `overdue-badge` and the `todo-card--overdue` class, while the future todo shows neither. Confirm completed todos with past due dates are NOT flagged.

### Tests for User Story 1

> **NOTE: Write these tests FIRST and ensure they FAIL before updating TodoCard.js**

- [ ] T005 [P] [US1] Extend `packages/frontend/src/components/__tests__/TodoCard.test.js` with overdue scenarios: renders `todo-card--overdue` class and `overdue-badge` for past-due incomplete todo; renders neither for completed past-due, no-due-date, and today/future-due todos (mock `getTodayString` to return a fixed date)

### Implementation for User Story 1

- [ ] T006 [US1] Update `packages/frontend/src/components/TodoCard.js` — import `isOverdue` and `getTodayString` from `../utils/overdueUtils`; compute `const overdue = isOverdue(todo, getTodayString())` inside the component; apply `todo-card--overdue` class to root element when overdue; render `<span className="overdue-badge">Overdue</span>` when overdue per the DOM structure in the UI contract

**Checkpoint**: User Story 1 is fully functional and independently testable. An incomplete todo with a past due date shows the color accent and "Overdue" badge; all non-overdue cases show no indicator.

---

## Phase 4: User Story 2 — Overdue Count Awareness (Priority: P2)

**Goal**: A `<div className="overdue-banner">` showing `"{n} overdue"` is rendered as the first child of the todo list when `overdueCount > 0`. The banner is completely absent when the count is zero.

**Independent Test**: Create three incomplete todos with past due dates, one completed past-due todo, and one future todo. Navigate to the list and confirm the banner reads "3 overdue". Complete one overdue todo and confirm the banner updates to "2 overdue". Complete all overdue todos and confirm the banner disappears entirely.

### Tests for User Story 2

> **NOTE: Write these tests FIRST and ensure they FAIL before updating TodoList.js**

- [ ] T007 [P] [US2] Extend `packages/frontend/src/components/__tests__/TodoList.test.js` with overdue banner scenarios: banner renders with correct count when multiple overdue todos exist; banner is absent when count is zero; completed past-due todos are excluded from the count; count updates correctly when a todo is toggled (mock `getTodayString` to return a fixed date)

### Implementation for User Story 2

- [ ] T008 [US2] Update `packages/frontend/src/components/TodoList.js` — import `isOverdue` and `getTodayString` from `../utils/overdueUtils`; compute `const todayStr = getTodayString()` and `const overdueCount = todos.filter(t => isOverdue(t, todayStr)).length` in the render body; conditionally render `<div className="overdue-banner">{overdueCount} overdue</div>` as the first child of the list container only when `overdueCount > 0`

**Checkpoint**: User Stories 1 AND 2 are both fully functional. The banner shows the correct live count and disappears when no overdue items exist.

---

## Phase 5: Polish & Cross-Cutting Concerns

**Purpose**: Final validation, coverage gate, and documentation verification.

- [ ] T009 Run the full frontend test suite (`npm test` inside `packages/frontend`) and confirm ≥ 80% line and branch coverage across `overdueUtils.js`, `TodoCard.js`, and `TodoList.js`
- [ ] T010 [P] Walk through all verification steps in `specs/001-overdue-todo-support/quickstart.md` to confirm the feature works end-to-end in the browser (start dev server, manually verify overdue badge, overdue banner, and zero-count hidden state)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately
- **Foundational (Phase 2)**: Depends on Phase 1 completion — **BLOCKS both user stories**
- **User Story 1 (Phase 3)**: Depends on Phase 2 completion (needs `overdueUtils` + CSS tokens)
- **User Story 2 (Phase 4)**: Depends on Phase 2 completion (needs `overdueUtils` + CSS tokens)
  - US1 and US2 are **independent** — they can be worked in parallel once Phase 2 is complete
- **Polish (Phase 5)**: Depends on both US1 and US2 being complete

### User Story Dependencies

- **User Story 1 (P1)**: No dependency on US2 — implements `TodoCard` only
- **User Story 2 (P2)**: No dependency on US1 — implements `TodoList` only
- Both stories consume `overdueUtils` (Foundational) and the CSS tokens (Foundational)

### Within Each User Story

- Tests MUST be written and confirmed FAILING before the corresponding implementation task
- Tests before implementation (T005 before T006; T007 before T008)
- T004 (CSS tokens) is fully independent and can run in parallel with T002/T003

### Parallel Opportunities

- T004 (CSS tokens) can run in parallel with T002 and T003 — completely separate file
- Once Phase 2 is complete, T005 and T007 (tests) can run in parallel across stories
- Once Phase 2 is complete, T006 and T008 (implementations) can run in parallel across stories
- T009 and T010 can run in parallel in the Polish phase

---

## Parallel Example: Phase 2 (Foundational)

```
# T002 and T004 can be started together by different developers:
Developer A: Write overdueUtils.test.js (T002)
Developer B: Add CSS tokens to theme.css (T004)

# Developer A then immediately follows with:
Developer A: Implement overdueUtils.js (T003) — after T002 tests are confirmed failing
```

## Parallel Example: User Stories (after Phase 2 is complete)

```
# US1 and US2 can be started together by different developers:
Developer A: Extend TodoCard.test.js (T005) → Update TodoCard.js (T006)
Developer B: Extend TodoList.test.js (T007) → Update TodoList.js (T008)
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (verify baseline)
2. Complete Phase 2: Foundational — T002 → T003, T004 in parallel
3. Complete Phase 3: User Story 1 — T005 → T006
4. **STOP and VALIDATE**: Confirm overdue badge and color are working independently
5. Merge/demo US1 as MVP

### Incremental Delivery

1. Setup + Foundational → `overdueUtils` + CSS tokens ready
2. Add User Story 1 (TodoCard) → visually distinct overdue items → **Demo/MVP**
3. Add User Story 2 (TodoList) → overdue count banner → **Full feature complete**
4. Each story adds value without breaking the previous one

### Parallel Team Strategy

With two developers:

1. Both complete Setup + Foundational together (or split T002/T003 from T004)
2. Once Foundational is complete:
   - Developer A: User Story 1 (T005 → T006, `TodoCard`)
   - Developer B: User Story 2 (T007 → T008, `TodoList`)
3. Both stories complete and integrate independently; Polish phase runs together

---

## Notes

- No backend files change — this feature is entirely frontend
- `isOverdue` is a pure function; always inject `todayStr` rather than calling `getTodayString()` inside it. This makes tests deterministic without mocking `Date`
- In component tests, mock `getTodayString` at the module level: `jest.mock('../utils/overdueUtils', () => ({ ...jest.requireActual('../utils/overdueUtils'), getTodayString: () => '2026-02-27' }))`
- Color tokens MUST be defined in `theme.css` — no raw hex values in component files
- The overdue badge MUST NOT have `aria-hidden` — its text content satisfies WCAG 1.4.1 naturally
- [P] tasks = different files, no blocking dependencies between them
- [Story] label maps each task to its user story for traceability (US1 = TodoCard; US2 = TodoList)
