<!--
SYNC IMPACT REPORT
==================
Version change: (none) → 1.0.0  [initial population from project docs]

Modified principles:
  - All five principles newly created (no prior named principles)

Added sections:
  - Core Principles (I–V)
  - Technology Stack
  - Development Workflow
  - Governance

Removed sections: none

Templates requiring updates:
  ✅ .specify/memory/constitution.md — written (this file)
  ✅ .specify/templates/plan-template.md — "Constitution Check" gate aligns with
     the five principles; no content changes required.
  ✅ .specify/templates/spec-template.md — FR/acceptance-scenario structure is
     consistent with functional-requirements discipline (Principle V); no changes
     required.
  ✅ .specify/templates/tasks-template.md — task phases and test-first structure
     align with Principle II (Test-First); no changes required.

Follow-up TODOs: none — all placeholders resolved.
-->

# Copilot Bootcamp Todo App Constitution

## Core Principles

### I. Clean Code & Consistent Style

All code MUST follow the project's formatting and naming conventions without exception:

- **Indentation**: 2 spaces for all file types (JS, JSON, CSS, Markdown).
- **Naming**: `camelCase` for variables/functions; `PascalCase` for React
  components and classes; `UPPER_SNAKE_CASE` for module-level constants.
- **Imports**: ordered external → internal → styles, separated by blank lines;
  no circular dependencies permitted.
- **Linting**: ESLint MUST pass with zero errors before every commit; warnings
  MUST be justified inline. `npm run lint` is a required pre-commit gate.
- **DRY**: duplicated logic MUST be extracted into shared utilities or hooks.
- **Comments**: explain *why*, never *what*; JSDoc MUST be provided for all
  exported functions and React components.

**Rationale**: Consistent style lowers cognitive load for AI-assisted and human
reviewers alike, and prevents lint noise from obscuring real defects.

### II. Test-First with 80%+ Coverage (NON-NEGOTIABLE)

Tests MUST be written before or alongside implementation; no feature is
considered complete without them:

- **Coverage target**: ≥ 80% line/branch coverage across every package,
  measured via Jest's coverage reporter.
- **Test isolation**: each test MUST set up its own data, mock all external
  dependencies (API calls, timers, localStorage), and clean up afterwards.
  Shared state between tests is forbidden.
- **File convention**: test files live in colocated `__tests__/` directories
  and MUST be named `{filename}.test.js`.
- **Scope**: unit tests MUST cover individual components and service functions;
  integration tests MUST cover component interactions and frontend↔backend API
  communication.
- **Quality over quantity**: tests MUST assert on *behavior*, not implementation
  details; brittle snapshot tests that break on minor refactors MUST be avoided.

**Rationale**: High, behavior-focused coverage is the primary safety net for
AI-generated code and refactoring sessions during the bootcamp.

### III. Single Responsibility & Layered Architecture

Every module, component, and function MUST have exactly one reason to change:

- React components MUST only handle presentation and local UI state. Data
  fetching, mutation, and business logic MUST live in service modules or custom
  hooks.
- Backend route handlers MUST delegate all business logic to the service layer;
  no SQL queries or domain rules in route files.
- A component that both displays and fetches data is a constitution violation.
- Props lists MUST be minimal and focused; unnecessary prop drilling MUST be
  resolved via composition or context, not by widening interfaces.

**Rationale**: Clear layer boundaries make AI-assisted code generation more
accurate and keep individual pieces independently testable.

### IV. Accessible & Themed UI (NON-NEGOTIABLE)

The interface MUST meet baseline usability and accessibility standards:

- **Color tokens**: all colors MUST reference the defined palette variables
  (light/dark modes); no raw hex values in component CSS unless defining the
  token itself.
- **Spacing grid**: all spacing values MUST use the 8px grid system (xs=8px,
  sm=16px, md=24px, lg=32px, xl=48px); arbitrary pixel values are forbidden.
- **Accessibility**: every interactive element MUST be keyboard accessible;
  color contrast MUST meet WCAG AA; icon buttons MUST have `aria-label` or
  `title` attributes.
- **Dark/light mode**: MUST be supported with the user's choice persisted in
  `localStorage`; system preference MUST be respected on first visit.
- **Responsive breakpoints**: mobile (<768px), tablet (768–1024px), and desktop
  (>1024px) breakpoints MUST all render a usable layout.

**Rationale**: Accessibility and design consistency are first-class
requirements, not post-hoc polish.

### V. Scope Discipline & Simplicity

The application MUST implement only what is specified in the functional
requirements; complexity MUST be justified:

- Features explicitly listed as *Out of Scope* (authentication, search,
  filtering, bulk operations, undo/redo, categories) MUST NOT be added without
  a formal requirements amendment.
- YAGNI: no abstractions, generics, or infrastructure should be built in
  anticipation of future requirements that are not currently specified.
- KISS: when two implementations satisfy the same requirement, the simpler one
  MUST be chosen. Complexity MUST be documented and justified.
- Premature performance optimisation (e.g., memoisation without a measured
  bottleneck) is a violation of this principle.

**Rationale**: A single-user todo app is intentionally simple; scope creep
dilutes bootcamp learning objectives and accumulates untested surface area.

## Technology Stack

The following stack is fixed for this project and MUST NOT be substituted
without a constitution amendment:

- **Frontend**: React (functional components + hooks), plain CSS (no external
  UI component libraries).
- **Backend**: Node.js + Express.js REST API.
- **Testing**: Jest for both packages; `@testing-library/react` for frontend
  component tests.
- **Package management**: npm workspaces (monorepo root manages both
  `packages/frontend` and `packages/backend`).
- **Linting**: ESLint with project-defined rules.

## Development Workflow

All contributors MUST follow this workflow before opening a pull request:

1. Run `npm run lint` from the repository root and resolve all errors.
2. Run `npm test` from the repository root and ensure all suites pass with
   ≥ 80% coverage.
3. Verify the Constitution Check section of the relevant `plan.md` before
   beginning implementation.
4. Request at least one peer code review before merging to `main`.
5. All todo data changes MUST be persisted to the backend immediately; no
   client-only state is acceptable as a final implementation.

## Governance

This constitution supersedes all other practices, verbal agreements, or
ad-hoc conventions. Amendments MUST:

1. Be proposed as a pull request editing `.specify/memory/constitution.md`.
2. Include updated version metadata following semantic versioning:
   - **MAJOR**: removal or backward-incompatible redefinition of a principle.
   - **MINOR**: new principle or section added, or materially expanded guidance.
   - **PATCH**: clarifications, wording fixes, or non-semantic refinements.
3. Be accompanied by updates to any affected template or documentation files.
4. Be approved by at least one reviewer before merging.

All PRs and AI-assisted code sessions MUST verify compliance with the five Core
Principles. Violations detected in review MUST be resolved before merge, not
deferred. Refer to `docs/coding-guidelines.md`, `docs/testing-guidelines.md`,
`docs/ui-guidelines.md`, and `docs/functional-requirements.md` for detailed
guidance supporting each principle.

**Version**: 1.0.0 | **Ratified**: 2026-02-27 | **Last Amended**: 2026-02-27
