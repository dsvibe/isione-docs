# Complete Task Flow

Records a [TaskEntry](../resources.md#taskentry) to mark a task as completed. Tasks can be completed multiple times, with each completion recorded as a separate entry.

## Components Involved

| Component | Role |
|-----------|------|
| web | User interface for task completion |
| android | NFC tag scanning for task completion |
| server | API endpoint and database operations |

## User Flow (Web)

### Immediate Completion

1. User views task list on HomePage
2. Clicks "Complete" button on a task
3. Confirmation dialog appears
4. User confirms completion
5. Task entry created with current timestamp
6. Task list refreshes showing updated completion time

### Backdated Completion

1. User clicks dropdown arrow next to "Complete" button
2. Selects "Record past completion..."
3. Datetime picker appears (max = current time)
4. User selects a past datetime
5. Clicks "Complete" to submit
6. Task entry created with specified timestamp
7. Task list refreshes

## User Flow (Android)

### NFC Tag Scan

1. User holds phone near an NFC tag linked to a task
2. App reads tag UUID from NDEF message (`isione://tag/{uuid}`)
3. App looks up tag via `GET /beta/nfc-tags/{uuid}`
4. App automatically creates task entry via `POST /entries`
5. Success dialog with haptic feedback confirms completion

### Unregistered Tag

If the tag isn't registered to a task:

1. App shows "Tag Not Registered" prompt
2. User taps "Register Tag"
3. User selects pod, then task
4. App registers tag via `POST /beta/pods/{podId}/nfc-tags`
5. Task entry is recorded automatically

## API Requests

### Create Task Entry

[Swagger: POST /beta/pods/{podId}/tasks/{taskId}/entries](https://api-docs.isione.dsvibe.io/#/tasks/completeTask)

```http
POST /beta/pods/{podId}/tasks/{taskId}/entries
Authorization: Bearer {token}
Content-Type: application/json

{
  "completedAt": "2024-01-15T10:30:00Z"
}
```

`completedAt` is optional. If omitted, server uses current timestamp.

**Response:**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440002",
  "taskId": "550e8400-e29b-41d4-a716-446655440001",
  "completedAt": "2024-01-15T10:30:00Z",
  "createdAt": "2024-01-15T10:30:05Z"
}
```

### Lookup NFC Tag

Used by Android to resolve a scanned tag UUID to its associated task.

```http
GET /beta/nfc-tags/{uuid}
Authorization: Bearer {token}
```

**Response (registered):**

```json
{
  "id": "...",
  "tagUuid": "019400f1-b1a2-7000-8000-000000000001",
  "taskId": "550e8400-e29b-41d4-a716-446655440001",
  "taskName": "Feed the cat",
  "podId": "550e8400-e29b-41d4-a716-446655440000",
  "podName": "My Household"
}
```

**Response (unregistered):** `404 Not Found`

### Register NFC Tag

Associates an unregistered tag with a task.

```http
POST /beta/pods/{podId}/nfc-tags
Authorization: Bearer {token}
Content-Type: application/json

{
  "tagUuid": "019400f1-b1a2-7000-8000-000000000001",
  "taskId": "550e8400-e29b-41d4-a716-446655440001"
}
```

**Response:** Same as Lookup NFC Tag.

## Sequence Diagrams

### Web

```
┌──────┐          ┌──────┐          ┌────────┐
│ User │          │ Web  │          │ Server │
└──┬───┘          └──┬───┘          └───┬────┘
   │  Click Complete │                  │
   │─────────────────>                  │
   │                 │                  │
   │  Confirm dialog │                  │
   │<─────────────────                  │
   │                 │                  │
   │  Confirm        │                  │
   │─────────────────>                  │
   │                 │  POST /entries   │
   │                 │─────────────────>│
   │                 │    201 Created   │
   │                 │<─────────────────│
   │                 │                  │
   │  Updated list   │ Refresh tasks    │
   │<─────────────────                  │
   │                 │                  │
```

### Android (NFC - Registered Tag)

```
┌──────┐          ┌─────────┐          ┌────────┐
│ User │          │ Android │          │ Server │
└──┬───┘          └────┬────┘          └───┬────┘
   │  Scan NFC tag     │                   │
   │──────────────────>│                   │
   │                   │ GET /nfc-tags/{uuid}
   │                   │──────────────────>│
   │                   │   NfcTagInfo      │
   │                   │<──────────────────│
   │                   │                   │
   │                   │ POST /entries     │
   │                   │──────────────────>│
   │                   │   201 Created     │
   │                   │<──────────────────│
   │                   │                   │
   │  Success + haptic │                   │
   │<──────────────────│                   │
   │                   │                   │
```

### Android (NFC - Unregistered Tag)

```
┌──────┐          ┌─────────┐          ┌────────┐
│ User │          │ Android │          │ Server │
└──┬───┘          └────┬────┘          └───┬────┘
   │  Scan NFC tag     │                   │
   │──────────────────>│                   │
   │                   │ GET /nfc-tags/{uuid}
   │                   │──────────────────>│
   │                   │      404          │
   │                   │<──────────────────│
   │                   │                   │
   │  "Not Registered" │                   │
   │<──────────────────│                   │
   │                   │                   │
   │  Tap "Register"   │                   │
   │──────────────────>│                   │
   │                   │ GET /pods         │
   │                   │──────────────────>│
   │                   │   Pod list        │
   │                   │<──────────────────│
   │                   │                   │
   │  Select pod/task  │                   │
   │──────────────────>│                   │
   │                   │ POST /nfc-tags    │
   │                   │──────────────────>│
   │                   │   NfcTagInfo      │
   │                   │<──────────────────│
   │                   │                   │
   │                   │ POST /entries     │
   │                   │──────────────────>│
   │                   │   201 Created     │
   │                   │<──────────────────│
   │                   │                   │
   │  Success + haptic │                   │
   │<──────────────────│                   │
   │                   │                   │
```

Completing a task affects its [status calculation](../task-status.md).