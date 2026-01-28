# Resources

Core domain concepts in IsiOne.

## Pod

A collaborative space where multiple users can work together to manage a shared set of tasks. Represents a household, team, or any group context.

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Auto-generated identifier |
| name | string | Human-readable name |
| createdAt | timestamp | Creation time |

## Task

A repeating action or chore that members of a pod need to complete. Tasks track timing for pending and overdue states.

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Auto-generated identifier |
| podId | UUID | Parent pod |
| name | string | Task name |
| displayText | string | Optional instructions |
| pendingAfter | interval | Duration until pending (microseconds) |
| overdueAfter | interval | Duration until overdue (microseconds) |
| createdAt | timestamp | Creation time |
| updatedAt | timestamp | Last modification time |

## TaskEntry

A record of a task completion. Each time a task is completed, a new entry is created.

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Auto-generated identifier |
| taskId | UUID | Parent task |
| userId | UUID | User who completed |
| completedAt | timestamp | When task was completed |
| createdAt | timestamp | When entry was recorded |
| updatedAt | timestamp | Last modification time |

## Membership

Links users to pods with role-based access.

| Field | Type | Description |
|-------|------|-------------|
| userId | UUID | User in the pod |
| podId | UUID | Pod they belong to |
| role | enum | `owner`, `member`, or `viewer` |
| createdAt | timestamp | When invited |
| acceptedAt | timestamp | When accepted (null = pending) |
