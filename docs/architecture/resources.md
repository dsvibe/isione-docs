# Resources

Core domain concepts in IsiOne.

## Pod

A collaborative space where multiple users can work together to manage a shared set of tasks. Represents a household, team, or any group context. Pods can mint per-pod [API keys](flows/api-keys.md) for device authentication — see [Authentication](authentication.md).

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

## API Key

A per-pod credential used by the physical controller to authenticate against the API. Each key is bound to a pod and to the user who created it: it authenticates only while that user holds an accepted membership for the pod. See [Authentication](authentication.md#api-keys-device-authentication) for the contract and [API Keys](flows/api-keys.md) for the lifecycle.

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Auto-generated identifier |
| name | string | Human-readable name (typically the device that holds it) |
| createdAt | timestamp | Creation time |
| lastUsed | timestamp | Last successful authentication, or null if never used |

The plaintext `key` is returned exactly once, in the response to creating the key, and is not included in subsequent reads.

Pod scope is implicit from the URL path. The creator binding affects key validity but is not exposed as a field on the resource.
