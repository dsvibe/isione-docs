# API Keys

API keys are per-pod credentials used by the physical controller to authenticate against the IsiOne API. The contract — header, key format, where keys are accepted, failure codes — lives on the [Authentication](../authentication.md) page. This page covers the operational flow: creating a key, handing it off to a device, and the two ways a key can end.

## Components Involved

| Component  | Role |
|------------|------|
| server     | API endpoints, key generation, silent revocation |
| web        | Pod-settings UI: create / list / delete keys; display the one-shot plaintext |
| controller | Authenticates against `GET /beta/pods/{podId}/device/status` using the key |

See also: [Device Slots](device-slots.md) for what the controller does with that endpoint and how slots are configured.

## Permissions

[Roles](../permissions.md#roles) and role-gated actions for this flow live on the [Permissions](../permissions.md#api-keys) page. The [Authentication methods](../permissions.md#authentication-methods) subsection records why an API-key-authenticated request can reach only a single endpoint.

## Lifecycle

A key is created via the pod-settings UI, returned in plaintext exactly once, then handed off to a device (typically the physical controller). It authenticates each subsequent request from that device against `GET /beta/pods/{podId}/device/status`.

A key reaches end-of-life in one of two ways:

- **Explicit deletion** via `DELETE /beta/pods/{podId}/api-keys/{keyId}`. Only the key's creator may delete it.
- **Silent revocation** when the creator's accepted pod membership is cleared — an owner removes them, or they leave. No `DELETE` is required; the next request from the device returns `401 "invalid API key"`. If the creator is later re-invited and accepts, their previously-created keys begin authenticating again, so delete them explicitly for permanent revocation.

## API Requests

[Swagger](https://api.isione.dsvibe.io/swagger/index.html) is the schema reference. The requests below are those that drive the flow.

### Create Key

[Swagger: POST /beta/pods/{podId}/api-keys](https://api.isione.dsvibe.io/swagger/index.html#/api-keys/post_beta_pods__podId__api-keys)

```http
POST /beta/pods/{podId}/api-keys
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Living-Room Controller"
}
```

Returns `201 Created` with `id`, `name`, `createdAt`, `lastUsed` (always `null` on creation), and the full plaintext `key`. The plaintext is only returned in this response — see [Authentication: one-shot plaintext](../authentication.md#one-shot-plaintext).

## Sequence Diagrams

### Create, hand off, and use

```
┌──────┐            ┌────────┐         ┌────────────┐
│ User │            │ Server │         │ Controller │
└──┬───┘            └───┬────┘         └─────┬──────┘
   │ POST /pods/{id}/api-keys              │
   │ {name: "Living-Room Controller"}      │
   │───────────────────▶│                  │
   │  201 Created       │                  │
   │  {id, name, createdAt,                │
   │   lastUsed: null, key: <plaintext>}   │
   │◀───────────────────│                  │
   │                    │                  │
   │ User pairs device with plaintext key  │
   │ ─────────────────────────────────────▶│
   │                    │                  │
   │                    │ GET /pods/{id}/device/status
   │                    │ X-API-Key: <key> │
   │                    │◀─────────────────│
   │                    │   200 OK         │
   │                    │ {slots: [...]}   │
   │                    │─────────────────▶│
```

### Silent revocation when creator is removed

```
┌───────┐           ┌────────┐         ┌────────────┐
│ Owner │           │ Server │         │ Controller │
└───┬───┘           └───┬────┘         └─────┬──────┘
    │ DELETE /pods/{id}/members/{creatorUserId}
    │──────────────────▶│                   │
    │   204 No Content  │                   │
    │◀──────────────────│                   │
    │                   │                   │
    │                   │ GET /pods/{id}/device/status
    │                   │ X-API-Key: <key>  │
    │                   │◀──────────────────│
    │                   │   401 Unauthorized
    │                   │ "invalid API key" │
    │                   │──────────────────▶│
```

## Operational Notes

### Rotating a key

There is no in-place rotation endpoint. To rotate: create a new key, re-pair the device with the new plaintext, then delete the old key.

### Retiring a single controller

[`DELETE /beta/pods/{podId}/api-keys/{keyId}`](https://api.isione.dsvibe.io/swagger/index.html#/api-keys/delete_beta_pods__podId__api-keys__keyId_) revokes one key without affecting any other key on the pod. Use this to retire a single controller while keeping others online. Only the key's creator may delete; non-creator members receive `403`.

### Removing a member who created keys

Removing a member silently revokes every key they created (see [Lifecycle](#lifecycle)). The rows persist but do not authenticate. For permanent revocation — for example, so a name is freed for reuse — also `DELETE` each key.

### Controller behaviour on 401

Treat the key as dead. Do not retry with the same key. Surface a "needs re-pairing" state in the device UI; the user creates a new key and re-pairs.

## Security

- The plaintext key is shown exactly once and is never recoverable from the API afterward. The pod-settings UI must treat the create response as sensitive — don't persist the plaintext to a recoverable client store beyond the moment of display or device-pairing.
- Sharing a single key between devices is technically supported but loses the per-device naming and `lastUsed` audit trail. Use one key per device.
