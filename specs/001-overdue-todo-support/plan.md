# Implementation Plan: Support for Overdue Todo Items

**Branch**: `001-overdue-todo-support` | **Date**: 2026-02-27 | **Spec**: [spec.md](spec.md)  
**Input**: Feature specification from `/specs/001-overdue-todo-support/spec.md`

## Summary

Add visual overdue indicators to the React todo list. A todo is overdue when it is incomplete and its due date is strictly before today's local date. Overdue items must be visually distinct using both a color change AND a non-color signal ("Overdue" badge or ⚠ icon, satisfying WCAG 1.4.1). An overdue count summary banner is shown at the top of the list (hidden when count is zero). Overdue status is derived at render time — no backend changes, no stored state, no timers.

## Technical Context

**Language/Version**: JavaScript (ES2020) — Node.js 18 (backend), React 18 (frontend)  
**Primary Dependencies**: React (functional components + hooks), Express.js; Jest + @testing-library/react  
**Storage**: In-memory array (backend todoService); no database. No storage changes needed for this feature.  
**Testing**: Jest (both packages); @testing-library/react for frontend components; ≥ 80% line/branch coverage required  
**Target Platform**: Single-page web application; modern evergreen browsers  
**Project Type**: Web application (React SPA + Express REST API monorepo)  
**Performance Goals**: No specific targets; overdue calculation is O(n) on the client list — negligible for a single-user app  
**Constraints**: Overdue detection uses `new Date()` at page-load time (user's local date); no background timers; no backend involvement  
**Scale/Scope**: Single-user todo app; all overdue logic lives entirely in the frontend

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| # | Principle | Status | Notes |
|---|-----------|--------|-------|
| I | Clean Code & Consistent Style | ✅ PASS | New utility function and component classes will follow camelCase/PascalCase; 2-space indent; JSDoc required on exported helpers |
| II | Test-First with 80%+ Coverage | ✅ PASS | Tests for `isOverdue()` utility and updated `TodoCard`/`TodoList` components must be written before or alongside code; coverage gate enforced |
| III | Single Responsibility & Layered Architecture | ✅ PASS | Overdue logic extracted to a pure utility function; `TodoCard` handles presentation only; no data-fetching added to display components |
| IV | Accessible & Themed UI | ✅ PASS | WCAG 1.4.1 satisfied by color + text/icon dual signal; color tokens from theme.css used; no raw hex values; keyboard accessibility unaffected |
| V | Scope Discipline & Simplicity | ✅ PASS | No sorting, filtering, or separate overdue view; no new backend endpoints; overdue status is derived, not stored |

**Gate result: ALL PASS — proceed to Phase 0.**

## Project Structure

### Documentation (this feature)

```text
specs/001-overdue-todo-support/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── overdue-ui-contract.md   # Phase 1 output
└── tasks.md             # Phase 2 output (NOT created by /speckit.plan)
```

### Source Code (affected paths)

```text
packages/frontend/
├── src/
│   ├── components/
│   │   ├── TodoCard.js           # Add overdue CSS class + "Overdue" badge
│   │   ├── TodoList.js           # Add overdue count summary banner
│   │   └── __tests__/
│   │       ├── TodoCard.test.js  # Extend with overdue scenarios
│   │       └── TodoList.test.js  # Extend with banner scenarios
│   └── utils/
│       ├── overdueUtils.js       # NEW — pure isOverdue(todo, today) helper
│       └── __tests__/
│           └── overdueUtils.test.js  # NEW — unit tests for helper
├── src/styles/
│   └── theme.css                 # Add --color-overdue-* tokens + .overdue-badge
│
packages/backend/                 # NO CHANGES REQUIRED
```

**Structure Decision**: Web application (Option 2). This feature is frontend-only; no backend files change. A new `utils/` directory is introduced in the frontend `src/` to house pure business-logic helpers, keeping components presentation-only (Principle III).

## Complexity Tracking

> No constitution violations — this section is informational only.

No added complexity. The feature is implemented entirely as derived UI state using a single pure utility function.
