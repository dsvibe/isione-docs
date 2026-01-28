# Create Task Flow

Creates a new [Task](../resources.md#task) within a pod.

## Components Involved

| Component | Role |
|-----------|------|
| web | User interface for task creation |
| server | API endpoint and database operations |

## User Flow (Web)

1. From HomePage, user clicks "New Task"
2. Navigates to `/tasks/new` (CreateTaskPage)
3. User enters:
   - Task name (required)
   - Description (optional)
   - Pending after duration (default: 1 day)
   - Overdue after duration (default: 2 days)
4. Clicks "Create Task" button
5. On success, navigates to HomePage with refreshed task list

## API Request

[Swagger: POST /beta/pods/{podId}/tasks](https://isione-api.home.demery.net/swagger/index.html#/tasks/post_beta_pods__podId__tasks)

```http
POST /beta/pods/{podId}/tasks
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Feed the cat",
  "displayText": "Don't forget wet food!",
  "pendingAfter": 86400000000,
  "overdueAfter": 172800000000
}
```

Durations are in [microseconds](../decisions.md#durations-in-microseconds).

## API Response

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "name": "Feed the cat",
  "displayText": "Don't forget wet food!",
  "pendingAfter": 86400000000,
  "overdueAfter": 172800000000
}
```

## Sequence Diagram

```
┌──────┐          ┌──────┐          ┌────────┐
│ User │          │ Web  │          │ Server │
└──┬───┘          └──┬───┘          └───┬────┘
   │  Click New Task │                  │
   │─────────────────>                  │
   │                 │                  │
   │  Fill form      │                  │
   │─────────────────>                  │
   │                 │                  │
   │  Submit         │                  │
   │─────────────────>                  │
   │                 │  POST /tasks     │
   │                 │─────────────────>│
   │                 │                  │ Verify membership
   │                 │                  │ Validate input
   │                 │                  │ Create task
   │                 │    201 Created   │
   │                 │<─────────────────│
   │                 │                  │
   │  Task list      │ Navigate home    │
   │<─────────────────                  │
   │                 │                  │
```

## Validation Rules

| Field | Rule | Error |
|-------|------|-------|
| name | Required, non-empty | "Name is required" |
| pendingAfter | Minimum 1 hour | "Pending duration too short" |
| overdueAfter | Minimum 1 hour | "Overdue duration too short" |
