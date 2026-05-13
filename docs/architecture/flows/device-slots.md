# Device Slots

A pod exposes six numbered slots that link tasks to physical positions on a controller. Pod members configure slot assignments through bearer-authenticated endpoints; a controller reads per-slot status through a dual-authentication endpoint that accepts either a bearer token or a per-pod API key. The controller's credential lifecycle lives on the [API Keys](api-keys.md) page; the dual-auth contract lives on the [Authentication](../authentication.md) page. This page covers the slot configuration flow and the read contract shared by the two callers.

## Components Involved

| Component  | Role |
|------------|------|
| server     | Stores slot assignments; computes per-slot status on every status request |
| web        | Pod-settings UI for assigning and clearing slots |
| controller | Polls `GET /beta/pods/{podId}/device/status` and renders LEDs from the response |

A bearer-authenticated client (web or mobile) may also call `GET /device/status` directly; the response shape is identical regardless of which credential authenticated the request.

## Permissions

[Roles](../permissions.md#roles) and role-gated actions live on the [Permissions](../permissions.md) page. The [Authentication methods](../permissions.md#authentication-methods) subsection records that `GET /device/status` is the one route that accepts an API key; every other route in this flow is bearer-only.

## Slot Model

A pod has exactly six slots, numbered `1` through `6`. The count is fixed and the slot numbers are stable: slot 3 means the same physical position to every controller paired to the pod and every member opening pod settings. Each slot is either assigned to exactly one task or unassigned.

Slot configuration is bound to the pod, not to any individual API key, so every controller paired to the pod sees the same assignments and slot state survives key rotation.

## Lifecycle

A pod member picks one of the pod's tasks in pod settings and assigns it to a slot via `PUT /device/slots/{slotNumber}`. The same endpoint clears a slot when the body's `taskId` is `null`. The controller polls `GET /device/status` on its own cadence and renders each slot's status as an LED pattern. Configuration and status are read through separate endpoints: the configuration view returns the raw `taskId` a member chose; the status view also resolves the assigned task's name and derives per-slot freshness from its [pending/overdue thresholds](../task-status.md).

## API Requests

[Swagger](https://api.isione.dsvibe.io/swagger/index.html) is the schema reference. The requests below are those that drive the flow.

### List Slots

```http
GET /beta/pods/{podId}/device/slots
Authorization: Bearer {token}
```

Returns `200 OK` with all six slots, ordered by slot number:

```json
{
  "slots": [
    {"slot": 1, "taskId": "550e8400-e29b-41d4-a716-446655440000"},
    {"slot": 2, "taskId": null},
    {"slot": 3, "taskId": null},
    {"slot": 4, "taskId": null},
    {"slot": 5, "taskId": null},
    {"slot": 6, "taskId": null}
  ]
}
```

A slot that has never been assigned and a slot that was previously assigned and then cleared appear identically — both as `{"slot": n, "taskId": null}`.

### Assign or Clear a Slot

```http
PUT /beta/pods/{podId}/device/slots/{slotNumber}
Authorization: Bearer {token}
Content-Type: application/json

{
  "taskId": "550e8400-e29b-41d4-a716-446655440000"
}
```

`slotNumber` must be in `1..6`; values outside that range return `400` before the body is read. Send `{"taskId": null}` to clear a slot. Assigning a task whose pod does not match the URL's `{podId}` returns `404 "task not found"` — there is no path through this endpoint to land a cross-pod task on a slot.

Returns `200 OK` with the updated slot:

```json
{"slot": 3, "taskId": "550e8400-e29b-41d4-a716-446655440000"}
```

### Get Status

```http
GET /beta/pods/{podId}/device/status
Authorization: Bearer {token}
```

Or, from a controller:

```http
GET /beta/pods/{podId}/device/status
X-API-Key: {key}
```

Returns `200 OK` with one entry per slot:

```json
{
  "slots": [
    {"slot": 1, "taskId": "550e8400-e29b-41d4-a716-446655440000", "taskName": "Feed the cat", "status": "pending"},
    {"slot": 2, "taskId": null, "taskName": null, "status": "unassigned"},
    {"slot": 3, "taskId": "...", "taskName": "Water plants", "status": "ok"},
    {"slot": 4, "taskId": "...", "taskName": "Take out bins", "status": "overdue"},
    {"slot": 5, "taskId": null, "taskName": null, "status": "unassigned"},
    {"slot": 6, "taskId": null, "taskName": null, "status": "unassigned"}
  ]
}
```

The dual-authentication contract — header, accepted format, failure codes — lives on the [Authentication](../authentication.md#where-each-is-accepted) page.

## Status Field

Each slot's `status` is one of:

| Value | Condition |
|-------|-----------|
| `unassigned` | No task is assigned to the slot |
| `ok` | Task is assigned and was completed within its `pendingAfter` window |
| `pending` | Time since the assigned task's latest completion is past `pendingAfter` but before `overdueAfter` |
| `overdue` | Time since the assigned task's latest completion is past `overdueAfter`, OR the task has never been completed |

Thresholds come from the assigned [Task](../resources.md#task)'s `pendingAfter` and `overdueAfter` durations and are evaluated server-side per request — the controller never interprets raw timestamps. The naming mirrors [Task Status](../task-status.md) with two differences: `ok` here corresponds to **Done** there, and `unassigned` is added for an empty slot.

**Never-completed reports as `overdue`.** A task that has just been assigned to a slot, with no completion history yet, returns `overdue` from the moment it appears on the device — consistent with the [Never Done](../task-status.md#never-done) rule. Clients that want to surface a brand-new task differently from a genuinely-late one must do so from the task's own data; the slot status alone does not distinguish them.

## Sequence Diagrams

### Configuring a slot from the web UI

```
┌──────┐                      ┌────────┐
│ User │                      │ Server │
└──┬───┘                      └───┬────┘
   │ GET /pods/{id}/tasks        │
   │────────────────────────────▶│
   │  200 OK                     │
   │  [{id, name, ...}, ...]     │
   │◀────────────────────────────│
   │                             │
   │ PUT /pods/{id}/device/slots/3
   │ {taskId: "<uuid>"}          │
   │────────────────────────────▶│
   │  200 OK                     │
   │  {slot: 3, taskId}          │
   │◀────────────────────────────│
   │                             │
   │ GET /pods/{id}/device/slots │
   │────────────────────────────▶│
   │  200 OK                     │
   │  {slots: [... updated ...]} │
   │◀────────────────────────────│
```

### Controller polling status

```
┌────────────┐                ┌────────┐
│ Controller │                │ Server │
└─────┬──────┘                └───┬────┘
      │ GET /pods/{id}/device/status
      │ X-API-Key: <key>         │
      │─────────────────────────▶│
      │  200 OK                  │
      │  {slots: [               │
      │    {slot, taskId,        │
      │     taskName, status},   │
      │    ... (6 entries)       │
      │  ]}                      │
      │◀─────────────────────────│
      │                          │
      │ ...wait poll interval... │
      │                          │
      │ GET /pods/{id}/device/status
      │ X-API-Key: <key>         │
      │─────────────────────────▶│
```

## Operational Notes

### Polling cadence

The polling cadence is set by the controller; the server enforces no rate limit on `GET /device/status` today. Each successful API-key-authenticated poll updates that key's [`lastUsed`](../authentication.md#lastused), so a quiet device remains observable from the pod's API-Keys list.

### Slots persist across key rotation

Slot configuration is per-pod, not per-key. The rotation procedure in [API Keys: Rotating a key](api-keys.md#rotating-a-key) — create a new key, re-pair the device, delete the old key — leaves every slot assignment untouched.

### No bulk slot endpoint

Reconfiguring all six slots takes six separate `PUT` requests. There is no bulk endpoint, and one is not planned.

### Controller behaviour on `unassigned`

Render the slot as inactive — no LED activity, blank label area. `unassigned` is the steady-state of an empty slot, not an error condition.

### Controller behaviour on `overdue`

Treat `overdue` uniformly; do not try to distinguish "just-assigned and never completed" from "completed long ago" at the controller. Any such distinction belongs to the user-facing clients that have access to the task's full history.

## Security

- Bearer requests to `/device/slots` and `/device/status` pass the same accepted-membership check as every other pod-scoped route — see [Permissions: Access tiers](../permissions.md#access-tiers).
- API-key requests are validated against the URL's `{podId}`. A key minted for pod A and used against pod B receives `403`.
- When the API key's creator loses their accepted pod membership, the next poll returns `401 "invalid API key"` even though both the key string and the slot configuration are unchanged. See the [API Keys lifecycle](api-keys.md#lifecycle) for the silent-revocation mechanism. Controllers must surface this as a re-pairing prompt, not a transient error.
