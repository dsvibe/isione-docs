# Invites & Membership Flow

Covers how users are invited to a [Pod](../resources.md#pod), how invites move between states, and how the resulting [Membership](../resources.md#membership) is managed.

A pod's creator becomes its `owner` at pod creation (see [Create Pod](create-pod.md)). All subsequent members and viewers join through this flow.

## Components Involved

| Component | Role |
|-----------|------|
| server | API endpoints, permission checks, state transitions |
| web | Consumes the invite API; no dedicated invites/members UI surface yet |

## Permissions

[Roles](../permissions.md#roles) and role-gated actions for this flow live in the cross-cutting [Permissions](../permissions.md#invites-membership) page. The [access tiers](../permissions.md#access-tiers) section there explains why a pending invitee cannot reach any pod-scoped route.

## State Machine

```
            invite created
                  │
                  ▼
              ┌────────┐  accept   ┌──────────┐
              │pending │──────────▶│ accepted │
              └────────┘           └──────────┘
                  │ reject              │ owner removes member
                  ▼                     ▼
              ┌────────┐           (row deleted)
              │rejected│
              └────────┘
```

From the API's perspective, a membership is observable in only one state at a time:

- **Pending** rows appear in `GET /beta/invites` (invitee's view) and `GET /beta/pods/{podId}/invites` (inviter's view).
- **Accepted** rows appear in `GET /beta/pods/{podId}/members`.
- **Rejected** rows are not listed by any endpoint but block re-invite — see below.

### Re-invite conflicts

`POST /beta/pods/{podId}/invites` returns `409 Conflict` when a row already exists for the target user:

| Existing state | Response message |
|----------------|------------------|
| accepted | `user is already a member` |
| rejected | `user has rejected a previous invite` |
| pending | `user already has a pending invite` |

The rejected state is **terminal**: there is no re-invite path through the API. Recovery requires a manual DB reset.

### Removal vs. rejection asymmetry

Member removal (`DELETE /beta/pods/{podId}/members/{userId}`) clears the membership entirely, so an owner can re-invite the same user afterward. Rejection marks the membership as rejected and blocks re-invite. The asymmetry is intentional: an invitee's "no" is sticky, an owner's "removed" is reversible.

## Auto-User Creation

Inviting an email that has no existing user creates one transparently — callers receive a `userId` for someone who has never logged in. The user's profile (name, last-login time, etc.) is populated when the invitee first authenticates via OIDC.

## Two Listing Endpoints

Listings are split by viewpoint:

- **`GET /beta/pods/{podId}/invites`** — *inviter's view*: "who have I invited to my pod?" Pod-scoped; requires accepted membership.
- **`GET /beta/invites`** — *invitee's view*: "what am I being invited to?" User-scoped; the only endpoint a pending invitee can call usefully. Each row includes `podName` and `invitedByName` so the invitee has context without pod access.

## API Requests

[Swagger](https://api-docs.isione.dsvibe.io/) is the schema reference. The requests below are those that drive the flow.

### Create Invite

[Swagger: POST /beta/pods/{podId}/invites](https://api-docs.isione.dsvibe.io/#/invites/createInvite)

```http
POST /beta/pods/{podId}/invites
Authorization: Bearer {token}
Content-Type: application/json

{
  "email": "user@example.com",
  "role": "member"
}
```

`role` must be `member` or `viewer`. The response carries `invitedBy` and `invitedByName` as nullable fields (null for legacy invites whose origin was never recorded, or when the inviter has no display name).

### Accept and Reject

The invitee acts on their pending invites:

```http
POST /beta/invites/{podId}/accept
POST /beta/invites/{podId}/reject
```

Acceptance gives the caller the role from the invite. Rejection is terminal — re-inviting the same email returns `409`; see [Re-invite conflicts](#re-invite-conflicts).

### Cancel and Remove

Two different undo paths with intentionally different semantics:

```http
DELETE /beta/pods/{podId}/invites/{userId}   # cancel a pending invite
DELETE /beta/pods/{podId}/members/{userId}   # remove an accepted member
```

- **Cancel** is authorised for the pod owner OR the original inviter (provided the inviter is still an accepted member). It only applies to pending invites; cancelling an accepted membership returns `400`.
- **Remove** is owner-only. The owner cannot remove themselves. The membership is cleared entirely, allowing re-invite — see [Removal vs. rejection asymmetry](#removal-vs-rejection-asymmetry).

### Listings

```http
GET /beta/invites                  # invitee's view across all pods
GET /beta/pods/{podId}/invites     # inviter's view: pending invites for one pod
GET /beta/pods/{podId}/members     # accepted members for one pod
```

See [Two Listing Endpoints](#two-listing-endpoints) for why the inviter and invitee have separate routes.

## Sequence Diagrams

### Invite, accept, member visible

```
┌─────────┐         ┌────────┐         ┌─────────┐
│ Inviter │         │ Server │         │ Invitee │
└────┬────┘         └───┬────┘         └────┬────┘
     │ POST /pods/{id}/invites             │
     │ {email, role}                       │
     │────────────────▶│                   │
     │                 │ Find or create    │
     │                 │ user by email     │
     │                 │ Create pending    │
     │                 │ invite            │
     │   201 Created   │                   │
     │◀────────────────│                   │
     │                 │                   │
     │                 │ GET /invites      │
     │                 │◀──────────────────│
     │                 │  pending invite   │
     │                 │──────────────────▶│
     │                 │                   │
     │                 │ POST /invites/{podId}/accept
     │                 │◀──────────────────│
     │                 │ Mark accepted     │
     │                 │   200 OK          │
     │                 │──────────────────▶│
     │                 │                   │
     │ GET /pods/{id}/members              │
     │────────────────▶│                   │
     │  invitee listed │                   │
     │◀────────────────│                   │
```

### Reject is terminal

```
┌─────────┐         ┌────────┐         ┌─────────┐
│ Inviter │         │ Server │         │ Invitee │
└────┬────┘         └───┬────┘         └────┬────┘
     │ POST /pods/{id}/invites             │
     │────────────────▶│                   │
     │   201 Created   │                   │
     │◀────────────────│                   │
     │                 │ POST /invites/{podId}/reject
     │                 │◀──────────────────│
     │                 │ Mark rejected     │
     │                 │   200 OK          │
     │                 │──────────────────▶│
     │                 │                   │
     │ POST /pods/{id}/invites             │
     │ (same email)    │                   │
     │────────────────▶│                   │
     │   409 Conflict  │                   │
     │ "user has rejected a previous invite"
     │◀────────────────│                   │
```

### Remove allows re-invite

```
┌───────┐           ┌────────┐         ┌────────┐
│ Owner │           │ Server │         │ Member │
└───┬───┘           └───┬────┘         └───┬────┘
    │ DELETE /pods/{id}/members/{userId}  │
    │──────────────────▶│                 │
    │                   │ Membership      │
    │                   │ cleared         │
    │   204 No Content  │                 │
    │◀──────────────────│                 │
    │                   │                 │
    │ POST /pods/{id}/invites             │
    │ (same email)      │                 │
    │──────────────────▶│                 │
    │   201 Created     │                 │
    │◀──────────────────│                 │
```

