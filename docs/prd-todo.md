# Product Requirements Document (PRD) - TODO App Upgrade

## 1. Overview

This upgrade makes the basic TODO app more useful for day-to-day task management while keeping the implementation simple and teachable for the lab. The MVP adds optional due dates, simple task priorities, date-based filters, and local-only persistence so users can identify urgent work without introducing backend changes or advanced workflow features.

---

## 2. MVP Scope

- Add an optional `dueDate` field to each task using ISO `YYYY-MM-DD` format.
- Add a `priority` field with enum values `P1`, `P2`, and `P3`.
- Default `priority` to `P3` when no value is provided.
- Add filter tabs for `All`, `Today`, and `Overdue`.
- In `All`, show both completed and incomplete tasks.
- In `Today` and `Overdue`, show only incomplete tasks.
- Keep storage local only, with no backend or external storage changes.
- Keep the existing app intentionally lean and suitable for instruction.
- Validate task inputs as follows:
- `title` is required.
- `priority` must be one of `P1`, `P2`, or `P3`.
- `dueDate` is optional.
- Invalid `dueDate` values should be ignored and treated as absent.

---

## 3. Post-MVP Scope

- Visually highlight overdue tasks in red so they stand out in the list.
- Display color-coded priority badges: red for `P1`, orange for `P2`, gray for `P3`.
- Apply advanced task sorting in this order: overdue first, then priority from `P1` to `P3`, then due date ascending, with tasks without a due date last.

---

## 4. Out of Scope

- Notifications.
- Recurring tasks.
- Multi-user support.
- Keyboard navigation enhancements.
- Additional accessibility work beyond the current baseline.
- Backend changes.
- External or cloud storage.