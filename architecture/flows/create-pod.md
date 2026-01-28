# Create Pod Flow

Creates a new [Pod](../resources.md#pod) and adds the authenticated user as owner.

## Components Involved

| Component | Role |
|-----------|------|
| web | User interface for pod creation |
| server | API endpoint and database operations |

## User Flow (Web)

1. User clicks "Create Pod" from the pod selector page
2. Navigates to `/pods/new` (CreatePodPage)
3. User enters pod name in the form
4. Clicks "Create Pod" button
5. On success, pod is selected and user returns to previous page

## API Request

[Swagger: POST /beta/pods](https://isione-api.home.demery.net/swagger/index.html#/pods/post_beta_pods)

```http
POST /beta/pods
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "My Household"
}
```

## API Response

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "My Household",
  "role": "owner",
  "pending": false
}
```

## Sequence Diagram

```
┌──────┐          ┌──────┐          ┌────────┐
│ User │          │ Web  │          │ Server │
└──┬───┘          └──┬───┘          └───┬────┘
   │  Click Create   │                  │
   │─────────────────>                  │
   │                 │                  │
   │  Enter name     │                  │
   │─────────────────>                  │
   │                 │                  │
   │  Submit form    │                  │
   │─────────────────>                  │
   │                 │  POST /beta/pods │
   │                 │─────────────────>│
   │                 │                  │ Create pod
   │                 │                  │ Create membership
   │                 │    201 Created   │
   │                 │<─────────────────│
   │                 │                  │
   │  Navigate home  │ Select pod       │
   │<─────────────────                  │
   │                 │                  │
```
