# Epics and Stories - TODO App Upgrade

> Based on requirements defined in [docs/prd-todo.md](./prd-todo.md).  
> Follows the structure defined in [docs/templates/epic-and-stories-template.md](./templates/epic-and-stories-template.md).

---

## Epics and Stories

### MVP

- Epic: Due Date Support
  - Story: Add `dueDate` field to the task data model
  - Story: Display due date on each task in the UI
  - Story: Allow user to set an optional due date when creating a task
  - Story: Ignore and treat invalid `dueDate` values as absent

- Epic: Task Priority
  - Story: Add `priority` field to the task data model
  - Story: Default `priority` to `P3` when no value is provided
  - Story: Allow user to select priority (`P1`, `P2`, `P3`) when creating a task
  - Story: Validate that `priority` is one of the accepted enum values

- Epic: Task Filtering
  - Story: Add filter tabs for `All`, `Today`, and `Overdue`
  - Story: Show all tasks (completed and incomplete) in the `All` tab
  - Story: Show only incomplete tasks in the `Today` tab
  - Story: Show only incomplete tasks in the `Overdue` tab

- Epic: Input Validation
  - Story: Require `title` when creating or editing a task
  - Story: Prevent task submission when required fields are missing

### Post-MVP

- Epic: Overdue Task Visual Highlighting
  - Story: Highlight overdue tasks in red in the task list

- Epic: Priority Badges
  - Story: Display a color-coded badge for each task's priority level
  - Story: Apply red badge for `P1`, orange for `P2`, gray for `P3`

- Epic: Advanced Task Sorting
  - Story: Sort overdue tasks to the top of the list
  - Story: Sort remaining tasks by priority (`P1` → `P3`)
  - Story: Sort tasks with the same priority by due date ascending
  - Story: Move tasks without a due date to the bottom of the list

---

## Acceptance Criteria

### MVP

#### Epic: Due Date Support

- Story: Add `dueDate` field to the task data model
  - AC: The task model includes a `dueDate` field in ISO `YYYY-MM-DD` format.
  - AC: The `dueDate` field is optional; tasks without a due date are valid.
- Story: Display due date on each task in the UI
  - AC: Each task displays its due date when one is set.
  - AC: Tasks without a due date show no date in the UI.
- Story: Allow user to set an optional due date when creating a task
  - AC: The task creation form includes a date input for due date.
  - AC: Submitting a task without a due date is allowed.
- Story: Ignore and treat invalid `dueDate` values as absent
  - AC: If a `dueDate` value is not a valid ISO `YYYY-MM-DD`, it is ignored.
  - AC: Tasks with invalid due dates behave as if no due date was set.

#### Epic: Task Priority

- Story: Add `priority` field to the task data model
  - AC: The task model includes a `priority` field with allowed values `P1`, `P2`, and `P3`.
- Story: Default `priority` to `P3` when no value is provided
  - AC: When no priority is selected, the task is saved with `priority: "P3"`.
- Story: Allow user to select priority (`P1`, `P2`, `P3`) when creating a task
  - AC: The task creation form includes a priority selector with options P1, P2, and P3.
  - AC: The selector defaults to P3.
- Story: Validate that `priority` is one of the accepted enum values
  - AC: Only `P1`, `P2`, or `P3` are accepted as valid priority values.
  - AC: Any value outside the enum is rejected or defaulted to `P3`.

#### Epic: Task Filtering

- Story: Add filter tabs for `All`, `Today`, and `Overdue`
  - AC: Three tabs are visible in the UI: All, Today, and Overdue.
  - AC: Clicking a tab updates the task list to show only the matching tasks.
- Story: Show all tasks (completed and incomplete) in the `All` tab
  - AC: The `All` tab displays both completed and incomplete tasks.
- Story: Show only incomplete tasks in the `Today` tab
  - AC: The `Today` tab displays only tasks with a due date equal to today's date.
  - AC: Completed tasks are not shown in the `Today` tab.
- Story: Show only incomplete tasks in the `Overdue` tab
  - AC: The `Overdue` tab displays only tasks with a due date in the past.
  - AC: Completed tasks are not shown in the `Overdue` tab.

#### Epic: Input Validation

- Story: Require `title` when creating or editing a task
  - AC: A task cannot be saved without a title.
  - AC: The user sees an error or the form is blocked when the title is empty.
- Story: Prevent task submission when required fields are missing
  - AC: The form prevents submission and surfaces validation errors when required fields are absent.

### Post-MVP

#### Epic: Overdue Task Visual Highlighting

- Story: Highlight overdue tasks in red in the task list
  - AC: Tasks with a due date before today are visually styled in red.
  - AC: Tasks that are not overdue use the default styling.

#### Epic: Priority Badges

- Story: Display a color-coded badge for each task's priority level
  - AC: Each task in the list displays a priority badge.
- Story: Apply red badge for `P1`, orange for `P2`, gray for `P3`
  - AC: P1 tasks display a red badge.
  - AC: P2 tasks display an orange badge.
  - AC: P3 tasks display a gray badge.

#### Epic: Advanced Task Sorting

- Story: Sort overdue tasks to the top of the list
  - AC: Tasks with a due date before today appear before all other tasks.
- Story: Sort remaining tasks by priority (`P1` → `P3`)
  - AC: Among non-overdue tasks, P1 tasks appear before P2, which appear before P3.
- Story: Sort tasks with the same priority by due date ascending
  - AC: Tasks with the same priority are ordered by due date, earliest first.
- Story: Move tasks without a due date to the bottom of the list
  - AC: Tasks without a due date appear after all tasks that have a due date.

---

## Technical Requirements

### MVP

#### Epic: Due Date Support

- Story: Add `dueDate` field to the task data model
  - TR: The `due_date DATE` column already exists in the SQLite `tasks` table in `packages/backend/src/app.js` — no schema change required.
  - TR: `POST /api/tasks` and `PUT /api/tasks/:id` already read `due_date` from the request body — no backend change required.
- Story: Display due date on each task in the UI
  - TR: `TaskList.js` already has a `formatDueDate()` helper that parses `task.due_date` and returns a localized string. Confirm the formatted date is rendered in each `ListItem`.
- Story: Allow user to set an optional due date when creating a task
  - TR: `TaskForm.js` already manages `dueDate` state and renders a MUI `TextField` of `type="date"` with `data-testid="due-date-input"`. No form changes required.
- Story: Ignore and treat invalid `dueDate` values as absent
  - TR: In `POST /api/tasks` and `PUT /api/tasks/:id` in `app.js`, add a regex guard (`/^\d{4}-\d{2}-\d{2}$/`) before storing `due_date`; set the column to `null` if the value fails validation.

#### Epic: Task Priority

- Story: Add `priority` field to the task data model
  - TR: Add `priority TEXT NOT NULL DEFAULT 'P3'` to the `CREATE TABLE tasks` statement in `app.js`.
- Story: Default `priority` to `P3` when no value is provided
  - TR: In the `POST /api/tasks` handler in `app.js`, resolve priority as `req.body.priority || 'P3'` before the INSERT statement. The SQLite column `DEFAULT 'P3'` also covers the case when the column is omitted entirely.
- Story: Allow user to select priority (`P1`, `P2`, `P3`) when creating a task
  - TR: Add a `priority` state variable (initial value `'P3'`) to `TaskForm.js`.
  - TR: Add a MUI `Select` (or `TextField` with `select`) rendering options P1, P2, P3 with `data-testid="priority-input"`.
  - TR: Include `priority` in the object passed to `onSave()` and reset it to `'P3'` on form clear.
  - TR: Sync `priority` from `initialTask.priority` in the `useEffect` that handles edit mode.
- Story: Validate that `priority` is one of the accepted enum values
  - TR: In `POST /api/tasks` and `PUT /api/tasks/:id` in `app.js`, validate that `priority` is in `['P1', 'P2', 'P3']`; default to `'P3'` if it is absent or invalid.

#### Epic: Task Filtering

- Story: Add filter tabs for `All`, `Today`, and `Overdue`
  - TR: Add a `filter` state variable (initial value `'All'`) in `App.js`.
  - TR: Render a MUI `Tabs` / `Tab` component above `TaskList` with values `All`, `Today`, and `Overdue`.
  - TR: Pass the active `filter` value as a prop to `TaskList`.
- Story: Show all tasks (completed and incomplete) in the `All` tab
  - TR: When `filter === 'All'`, call `GET /api/tasks` with no additional query parameters so both completed and incomplete tasks are returned.
- Story: Show only incomplete tasks in the `Today` tab
  - TR: Extend the `buildTaskQuery` helper in `app.js` to handle a `filter=today` query param, adding `WHERE due_date = DATE('now') AND completed = 0` to the query.
  - TR: When `filter === 'Today'`, `TaskList.js` calls `GET /api/tasks?filter=today`.
- Story: Show only incomplete tasks in the `Overdue` tab
  - TR: Extend `buildTaskQuery` in `app.js` to handle `filter=overdue`, adding `WHERE due_date < DATE('now') AND completed = 0`.
  - TR: When `filter === 'Overdue'`, `TaskList.js` calls `GET /api/tasks?filter=overdue`.

#### Epic: Input Validation

- Story: Require `title` when creating or editing a task
  - TR: `TaskForm.js` already guards with `if (!title.trim()) { setError('Title is required'); return; }` — no frontend change required.
  - TR: `POST /api/tasks` and `PUT /api/tasks/:id` in `app.js` already return HTTP 400 with `{ error: 'Task title is required' }` when title is missing — no backend change required.
- Story: Prevent task submission when required fields are missing
  - TR: The MUI `TextField` for title already has the `required` attribute, which triggers browser-level validation. The `handleSubmit` guard in `TaskForm.js` provides a second layer of enforcement and renders the error message via the existing `{error && <Typography color="error">}` block.

### Post-MVP

#### Epic: Overdue Task Visual Highlighting

- Story: Highlight overdue tasks in red in the task list
  - TR: In `TaskList.js`, compute `isOverdue` per task: compare `task.due_date` as a local date against today using `new Date(year, month - 1, day) < today`, only for incomplete tasks.
  - TR: When `isOverdue` is true, override the `ListItem` `sx` `borderColor` and `background` to red-tinted values (e.g., `rgba(211, 47, 47, 0.1)` background, `rgba(211, 47, 47, 0.4)` border).

#### Epic: Priority Badges

- Story: Display a color-coded badge for each task's priority level
  - TR: In `TaskList.js`, render a MUI `Chip` inside each `ListItem` displaying `task.priority`.
- Story: Apply red badge for `P1`, orange for `P2`, gray for `P3`
  - TR: Map priority to MUI `Chip` color: `P1 → 'error'` (red), `P2 → 'warning'` (orange), `P3 → 'default'` (gray). Apply via the `color` prop on the `Chip` component.

#### Epic: Advanced Task Sorting

- Story: Sort overdue tasks to the top of the list
  - TR: Add a `sortTasks(tasks)` utility function in `TaskList.js`. In the sort comparator, assign overdue tasks a group value of `0`, all others `1`, and sort ascending by group.
- Story: Sort remaining tasks by priority (`P1` → `P3`)
  - TR: In `sortTasks`, map priority to a numeric weight: `{ P1: 1, P2: 2, P3: 3 }`. Use this as the secondary sort key after the overdue group.
- Story: Sort tasks with the same priority by due date ascending
  - TR: Use `due_date` string comparison (ISO format sorts lexicographically) as the tertiary sort key in `sortTasks`.
- Story: Move tasks without a due date to the bottom of the list
  - TR: In `sortTasks`, treat a missing `due_date` as `'9999-99-99'` for comparison purposes so undated tasks always sort last.
  - Story: Display due date on each task in the UI
    - AC: Each task displays its due date when one is set.
    - AC: Tasks without a due date show no date in the UI.
    - TR: `TaskList.js` already has a `formatDueDate()` helper that parses `task.due_date` and returns a localized string. Confirm the formatted date is rendered in each `ListItem`.
  - Story: Allow user to set an optional due date when creating a task
    - AC: The task creation form includes a date input for due date.
    - AC: Submitting a task without a due date is allowed.
    - TR: `TaskForm.js` already manages `dueDate` state and renders a MUI `TextField` of `type="date"` with `data-testid="due-date-input"`. No form changes required.
  - Story: Ignore and treat invalid `dueDate` values as absent
    - AC: If a `dueDate` value is not a valid ISO `YYYY-MM-DD`, it is ignored.
    - AC: Tasks with invalid due dates behave as if no due date was set.
    - TR: In `POST /api/tasks` and `PUT /api/tasks/:id` in `app.js`, add a regex guard (`/^\d{4}-\d{2}-\d{2}$/`) before storing `due_date`; set the column to `null` if the value fails validation.

- Epic: Task Priority
  - Story: Add `priority` field to the task data model
    - AC: The task model includes a `priority` field with allowed values `P1`, `P2`, and `P3`.
    - TR: Add `priority TEXT NOT NULL DEFAULT 'P3'` to the `CREATE TABLE tasks` statement in `app.js`.
  - Story: Default `priority` to `P3` when no value is provided
    - AC: When no priority is selected, the task is saved with `priority: "P3"`.
    - TR: In the `POST /api/tasks` handler in `app.js`, resolve priority as `req.body.priority || 'P3'` before the INSERT statement. The SQLite column `DEFAULT 'P3'` also covers the case when the column is omitted entirely.
  - Story: Allow user to select priority (`P1`, `P2`, `P3`) when creating a task
    - AC: The task creation form includes a priority selector with options P1, P2, and P3.
    - AC: The selector defaults to P3.
    - TR: Add a `priority` state variable (initial value `'P3'`) to `TaskForm.js`.
    - TR: Add a MUI `Select` (or `TextField` with `select`) rendering options P1, P2, P3 with `data-testid="priority-input"`.
    - TR: Include `priority` in the object passed to `onSave()` and reset it to `'P3'` on form clear.
    - TR: Sync `priority` from `initialTask.priority` in the `useEffect` that handles edit mode.
  - Story: Validate that `priority` is one of the accepted enum values
    - AC: Only `P1`, `P2`, or `P3` are accepted as valid priority values.
    - AC: Any value outside the enum is rejected or defaulted to `P3`.
    - TR: In `POST /api/tasks` and `PUT /api/tasks/:id` in `app.js`, validate that `priority` is in `['P1', 'P2', 'P3']`; default to `'P3'` if it is absent or invalid.

- Epic: Task Filtering
  - Story: Add filter tabs for `All`, `Today`, and `Overdue`
    - AC: Three tabs are visible in the UI: All, Today, and Overdue.
    - AC: Clicking a tab updates the task list to show only the matching tasks.
    - TR: Add a `filter` state variable (initial value `'All'`) in `App.js`.
    - TR: Render a MUI `Tabs` / `Tab` component above `TaskList` with values `All`, `Today`, and `Overdue`.
    - TR: Pass the active `filter` value as a prop to `TaskList`.
  - Story: Show all tasks (completed and incomplete) in the `All` tab
    - AC: The `All` tab displays both completed and incomplete tasks.
    - TR: When `filter === 'All'`, call `GET /api/tasks` with no additional query parameters so both completed and incomplete tasks are returned.
  - Story: Show only incomplete tasks in the `Today` tab
    - AC: The `Today` tab displays only tasks with a due date equal to today's date.
    - AC: Completed tasks are not shown in the `Today` tab.
    - TR: Extend the `buildTaskQuery` helper in `app.js` to handle a `filter=today` query param, adding `WHERE due_date = DATE('now') AND completed = 0` to the query.
    - TR: When `filter === 'Today'`, `TaskList.js` calls `GET /api/tasks?filter=today`.
  - Story: Show only incomplete tasks in the `Overdue` tab
    - AC: The `Overdue` tab displays only tasks with a due date in the past.
    - AC: Completed tasks are not shown in the `Overdue` tab.
    - TR: Extend `buildTaskQuery` in `app.js` to handle `filter=overdue`, adding `WHERE due_date < DATE('now') AND completed = 0`.
    - TR: When `filter === 'Overdue'`, `TaskList.js` calls `GET /api/tasks?filter=overdue`.

- Epic: Input Validation
  - Story: Require `title` when creating or editing a task
    - AC: A task cannot be saved without a title.
    - AC: The user sees an error or the form is blocked when the title is empty.
    - TR: `TaskForm.js` already guards with `if (!title.trim()) { setError('Title is required'); return; }` — no frontend change required.
    - TR: `POST /api/tasks` and `PUT /api/tasks/:id` in `app.js` already return HTTP 400 with `{ error: 'Task title is required' }` when title is missing — no backend change required.
  - Story: Prevent task submission when required fields are missing
    - AC: The form prevents submission and surfaces validation errors when required fields are absent.
    - TR: The MUI `TextField` for title already has the `required` attribute, which triggers browser-level validation. The `handleSubmit` guard in `TaskForm.js` provides a second layer of enforcement and renders the error message via the existing `{error && <Typography color="error">}` block.

---

## Post-MVP

- Epic: Overdue Task Visual Highlighting
  - Story: Highlight overdue tasks in red in the task list
    - AC: Tasks with a due date before today are visually styled in red.
    - AC: Tasks that are not overdue use the default styling.
    - TR: In `TaskList.js`, compute `isOverdue` per task: compare `task.due_date` as a local date against today using `new Date(year, month - 1, day) < today`, only for incomplete tasks.
    - TR: When `isOverdue` is true, override the `ListItem` `sx` `borderColor` and `background` to red-tinted values (e.g., `rgba(211, 47, 47, 0.1)` background, `rgba(211, 47, 47, 0.4)` border).

- Epic: Priority Badges
  - Story: Display a color-coded badge for each task's priority level
    - AC: Each task in the list displays a priority badge.
    - TR: In `TaskList.js`, render a MUI `Chip` inside each `ListItem` displaying `task.priority`.
  - Story: Apply red badge for `P1`, orange for `P2`, gray for `P3`
    - AC: P1 tasks display a red badge.
    - AC: P2 tasks display an orange badge.
    - AC: P3 tasks display a gray badge.
    - TR: Map priority to MUI `Chip` color: `P1 → 'error'` (red), `P2 → 'warning'` (orange), `P3 → 'default'` (gray). Apply via the `color` prop on the `Chip` component.

- Epic: Advanced Task Sorting
  - Story: Sort overdue tasks to the top of the list
    - AC: Tasks with a due date before today appear before all other tasks.
    - TR: Add a `sortTasks(tasks)` utility function in `TaskList.js`. In the sort comparator, assign overdue tasks a group value of `0`, all others `1`, and sort ascending by group.
  - Story: Sort remaining tasks by priority (`P1` → `P3`)
    - AC: Among non-overdue tasks, P1 tasks appear before P2, which appear before P3.
    - TR: In `sortTasks`, map priority to a numeric weight: `{ P1: 1, P2: 2, P3: 3 }`. Use this as the secondary sort key after the overdue group.
  - Story: Sort tasks with the same priority by due date ascending
    - AC: Tasks with the same priority are ordered by due date, earliest first.
    - TR: Use `due_date` string comparison (ISO format sorts lexicographically) as the tertiary sort key in `sortTasks`.
  - Story: Move tasks without a due date to the bottom of the list
    - AC: Tasks without a due date appear after all tasks that have a due date.
    - TR: In `sortTasks`, treat a missing `due_date` as `'9999-99-99'` for comparison purposes so undated tasks always sort last.
