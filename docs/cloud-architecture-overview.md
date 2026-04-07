# Cloud Architecture Overview

This document describes the high-level architecture of the TODO App monorepo using system context and sequence diagrams.

---

## System Context Diagram

The following diagram shows the major components of the system and how they relate to each other.

```mermaid
graph TD
    User["👤 User\n(Browser)"]
    Frontend["React Frontend\npackages/frontend\n(Material UI, port 3000)"]
    Backend["Express API\npackages/backend\n(Node.js, port 3001)"]
    Store["In-Memory Store\nSQLite :memory:\n(better-sqlite3)"]

    User -->|"Interacts via browser"| Frontend
    Frontend -->|"REST API calls\n/api/tasks"| Backend
    Backend -->|"SQL queries"| Store
    Store -->|"Query results"| Backend
    Backend -->|"JSON responses"| Frontend
    Frontend -->|"Renders UI"| User
```

---

## Sequence Diagram: User Creates a TODO

The following diagram shows the end-to-end flow when a user fills in the task form and submits a new task.

```mermaid
sequenceDiagram
    actor User
    participant UI as React Frontend
    participant API as Express API
    participant DB as In-Memory SQLite

    User->>UI: Fill in task form (title, priority, dueDate)
    User->>UI: Click "Add Task"
    UI->>UI: Validate title (required field check)
    UI->>API: POST /api/tasks\n{ title, description, due_date, priority }
    API->>API: Validate title (required)\nValidate priority enum\nValidate due_date format
    API->>DB: INSERT INTO tasks\n(title, description, due_date, priority)
    DB-->>API: lastInsertRowid
    API->>DB: SELECT * FROM tasks WHERE id = ?
    DB-->>API: New task row
    API-->>UI: 201 Created { task }
    UI->>UI: Increment refreshKey to trigger re-fetch
    UI->>API: GET /api/tasks
    API->>DB: SELECT * FROM tasks\nORDER BY due_date, created_at
    DB-->>API: Task list
    API-->>UI: 200 OK [ ...tasks ]
    UI-->>User: Updated task list displayed
```
