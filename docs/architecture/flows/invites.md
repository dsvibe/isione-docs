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
            (owner/member)
                  │
                  ▼
              ┌────────┐  accept   ┌──────────┐
              │pending │──────────▶│ accepted │
              └────────┘           └──────────┘
               │      │                  │
        reject │      │ cancel           │ owner removes member
               │      │ (owner/inviter)  │
               ▼      ▼                  ▼
          ┌────────┐ (row deleted) ─── (row deleted)
          │rejected│ re-invitable      re-invitable
          └────────┘
           sticky:
           blocks re-invite
```

Both **cancel** (withdrawing a pending invite) and **remove** (ejecting an accepted member) delete the row, leaving the user re-invitable. Only **reject** persists, blocking re-invite. See [Cancel vs. reject vs. remove](#cancel-vs-reject-vs-remove) for the full asymmetry.

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

The rejected state is **terminal**: there is no re-invite path and no API flow to clear a rejection — see [Cancel vs. reject vs. remove](#cancel-vs-reject-vs-remove).

### Cancel vs. reject vs. remove

Three different "undo" actions exist, and they are deliberately **not** symmetric. This is the part consumers most often get wrong.

> **Mental model:** an invitee's refusal is permanent; an owner's withdrawal is always reversible.

| Action | Who | Applies to | Effect | Re-invitable after? |
|--------|-----|------------|--------|:-------------------:|
| **Reject** | invitee | pending invite | marks the membership `rejected` | **No** — sticky |
| **Cancel** | owner or original inviter | pending invite | deletes the row | Yes |
| **Remove** | owner | accepted member | deletes the row | Yes |

- **Reject** (`POST /beta/invites/{podId}/reject`): the invitee says no. The membership becomes `rejected` and re-invite returns `409`. There is no server flow to clear a rejection, so today it is effectively permanent.
- **Cancel** (`DELETE /beta/pods/{podId}/invites/{userId}`): the owner or original inviter withdraws a *pending* invite. The row is deleted outright, so the same user can be invited again (`201`). Attempting to cancel an *accepted* membership through this endpoint returns `400` ("cannot cancel accepted membership") — use the member-removal endpoint instead.
- **Remove** (`DELETE /beta/pods/{podId}/members/{userId}`): the owner ejects an *accepted* member. The row is deleted, so the user can be re-invited and re-accept later.

Remove is the intentional asymmetric counterpart to rejection: an invitee's "no" sticks, but an owner's "removed" does not.

## Auto-User Creation

Inviting an email that has no existing user creates one transparently — callers receive a `userId` for someone who has never logged in. The user's profile (name, last-login time, etc.) is populated when the invitee first authenticates via OIDC.

## Two Listing Endpoints

Listings are split by viewpoint:

- **`GET /beta/pods/{podId}/invites`** — *inviter's view*: "who have I invited to my pod?" Pod-scoped; requires accepted membership.
- **`GET /beta/invites`** — *invitee's view*: "what am I being invited to?" User-scoped; the only endpoint a pending invitee can call usefully. Each row includes `podName` and `invitedByName` so the invitee has context without pod access.

## API Requests

[Swagger](https://api-docs.isione.dsvibe.io/) is the schema reference for all endpoints, parameters, and status codes. The requests below are those that drive the flow.

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

`role` must be `member` or `viewer` — the `owner` role is **not assignable via an invite** (there is no ownership-transfer flow; see [Open questions](#open-questions)). Inviting an email with no existing user [auto-creates one](#auto-user-creation). The created invite echoes back `invitedBy` and `invitedByName` identifying the sender. In the listing views these fields can be null — for legacy invites whose origin predates inviter tracking, or when the inviter has no display name.

### Accept and Reject

[Swagger: POST /beta/invites/{podId}/accept](https://api-docs.isione.dsvibe.io/#/invites/acceptInvite)
· [Swagger: POST /beta/invites/{podId}/reject](https://api-docs.isione.dsvibe.io/#/invites/rejectInvite)

The invitee acts on their own pending invites:

```http
POST /beta/invites/{podId}/accept
POST /beta/invites/{podId}/reject
```

Acceptance gives the caller the role from the invite. Both endpoints require a *pending* invite to act on:

- No invite to act on (no pending invite for the caller on that pod) → `404`.
- The invite is no longer pending — already accepted, or already rejected → `400`.

Rejection is terminal — re-inviting the same email returns `409`; see [Re-invite conflicts](#re-invite-conflicts).

### Cancel and Remove

[Swagger: DELETE /beta/pods/{podId}/invites/{userId}](https://api-docs.isione.dsvibe.io/#/invites/deleteInvite)
· [Swagger: DELETE /beta/pods/{podId}/members/{userId}](https://api-docs.isione.dsvibe.io/#/members/removePodMember)

Two different undo paths with intentionally different semantics:

```http
DELETE /beta/pods/{podId}/invites/{userId}   # cancel a pending invite
DELETE /beta/pods/{podId}/members/{userId}   # remove an accepted member
```

- **Cancel** is authorised for the pod owner OR the original inviter (provided the inviter is still an accepted member). It only applies to pending invites; cancelling an accepted membership returns `400` (use removal instead). Success is `204`.
- **Remove** is owner-only. The owner cannot remove themselves (`400`). The membership is cleared entirely, allowing re-invite. Success is `204`.

See [Cancel vs. reject vs. remove](#cancel-vs-reject-vs-remove) for why these two deletes and rejection behave differently.

### Listings

[Swagger: GET /beta/invites](https://api-docs.isione.dsvibe.io/#/invites/listMyInvites)
· [Swagger: GET /beta/pods/{podId}/invites](https://api-docs.isione.dsvibe.io/#/invites/listPodInvites)
· [Swagger: GET /beta/pods/{podId}/members](https://api-docs.isione.dsvibe.io/#/members/listPodMembers)

```http
GET /beta/invites                  # invitee's view across all pods
GET /beta/pods/{podId}/invites     # inviter's view: pending invites for one pod
GET /beta/pods/{podId}/members     # accepted members for one pod
```

See [Two Listing Endpoints](#two-listing-endpoints) for why the inviter and invitee have separate routes.

## Open questions

Gaps in the current server surface, flagged for product confirmation:

- **No way to clear a rejection.** Rejection is permanent today — there is no API flow to reset a `rejected` membership, so a user who declines can never be re-invited. Confirm whether this is intended.
- **No ownership transfer.** The `owner` role is granted only at pod creation and cannot be assigned via invite or reassigned afterward.
- **No way for an owner to leave a pod they own.** An owner cannot remove themselves (`400`), and with no transfer flow there is no documented path for an owner to exit.

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

