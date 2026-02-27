# Feature Specification: Support for Overdue Todo Items

**Feature Branch**: `001-overdue-todo-support`  
**Created**: 2026-02-27  
**Status**: Draft  
**Input**: User description: "Support for Overdue Todo Items"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Visual Overdue Indicators (Priority: P1)

A user views their todo list and can immediately tell which tasks are overdue without manually comparing due dates to today's date. Overdue items are visually distinct from items that are on-time or have no due date, enabling instant prioritization.

**Why this priority**: This is the core value of the feature. Visual distinction is what lets users act immediately on overdue items. Without it, all other enhancements have no foundation.

**Independent Test**: Can be fully tested by creating a todo with a due date in the past, then viewing the todo list, and confirming the item appears visually distinct (e.g., highlighted, colored differently, or labeled "Overdue") compared to a todo with a future due date.

**Acceptance Scenarios**:

1. **Given** an incomplete todo with a due date that has already passed, **When** the user views the todo list, **Then** that todo is visually distinguished from non-overdue items (e.g., highlighted, colored differently, or labeled "Overdue").
2. **Given** an incomplete todo with a due date of today, **When** the user views the todo list, **Then** that todo is NOT displayed as overdue.
3. **Given** an incomplete todo with no due date, **When** the user views the todo list, **Then** that todo is NOT displayed as overdue.
4. **Given** a completed todo with a due date in the past, **When** the user views the todo list, **Then** that todo is NOT displayed as overdue.

---

### User Story 2 - Overdue Count Awareness (Priority: P2)

A user can immediately see a summary count of overdue todos somewhere prominent in the UI (e.g., a badge or count indicator), so they understand their overdue workload at a glance without scrolling through the full list.

**Why this priority**: Provides at-a-glance awareness of workload urgency. Less critical than the core visual indicators but meaningfully improves the user's ability to prioritize.

**Independent Test**: Can be fully tested by creating multiple todos with past due dates, navigating to the todo list, and confirming the overdue count indicator matches the number of incomplete, overdue todos.

**Acceptance Scenarios**:

1. **Given** three incomplete todos with past due dates, **When** the user views the todo list, **Then** an overdue count indicator displays "3 overdue".
2. **Given** no incomplete todos with past due dates, **When** the user views the todo list, **Then** the overdue count indicator is either hidden or displays "0 overdue".
3. **Given** one completed todo with a past due date and one incomplete todo with a past due date, **When** the user views the todo list, **Then** the overdue count indicator displays "1 overdue".

---

### Edge Cases

- What happens when a todo's due date is exactly today — is it overdue or on time? (Assumption: due today is NOT overdue; overdue means strictly before today.)
- How does the system handle timezone differences that may cause a due date to appear as past or future depending on the user's local time? (Assumption: comparison uses the user's local date as reported by their browser/device.)
- What happens when a todo is marked complete while it is displayed as overdue — does the overdue indicator disappear immediately? (Expected: yes, completing a task removes its overdue status immediately.)
- What happens if a user edits an overdue todo's due date to a future date — does the overdue indicator disappear? (Expected: yes, overdue status refreshes based on the updated due date.)

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST visually distinguish incomplete todos whose due date is before today's date from all other todos in the list.
- **FR-002**: The system MUST NOT mark a todo as overdue if its due date is today or in the future.
- **FR-003**: The system MUST NOT mark a completed todo as overdue, regardless of its due date.
- **FR-004**: The system MUST NOT mark a todo with no due date as overdue.
- **FR-005**: The overdue visual indicator MUST be removed immediately when a todo is marked as complete.
- **FR-006**: The overdue visual indicator MUST be removed immediately when a todo's due date is updated to today or a future date.
- **FR-007**: The system MUST display a count of currently overdue (incomplete, past-due) todos to the user.
- **FR-008**: The overdue count MUST update immediately when the count changes (e.g., a todo is completed, deleted, or its due date is changed).
- **FR-009**: Overdue detection MUST be based on the user's local date, comparing the todo's due date to today's date.

### Key Entities

- **Todo Item**: An existing entity with a title, optional due date, and completion status. Overdue status is derived — not stored — by comparing due date to today's local date at display time.
- **Overdue Status**: A derived property of a Todo Item. A todo is overdue when: due date is set AND due date is before today's date AND completion status is incomplete.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can identify all overdue tasks within 5 seconds of opening the todo list, without reading any dates.
- **SC-002**: 100% of incomplete todos with a due date before today are visually flagged as overdue when viewed.
- **SC-003**: 100% of completed todos are never flagged as overdue, regardless of their due date.
- **SC-004**: The overdue count displayed to the user is accurate at all times and reflects real-time changes as todos are completed, edited, or deleted.
- **SC-005**: Users report being able to prioritize their work more effectively as a result of the overdue indicators (qualitative improvement in task prioritization).

## Assumptions

- Overdue calculation is performed on the client side using the user's local date; no server-side scheduled job or background process is required.
- "Overdue" means strictly before today (yesterday or earlier). Due today is considered on time.
- This feature does not require changes to the data model — overdue status is derived at display time.
- Completed todos are excluded from overdue status to avoid confusion and noise in the UI.
- The feature applies to the existing single-user todo list — no filtering, sorting, or separate "overdue" view is required (overdue items remain in the main list, just visually highlighted).
