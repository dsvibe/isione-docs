# Permissions

Single source of truth for role-gated actions across the API. Each row names the action, the roles that can perform it, and links to the flow doc that exercises it. If you add a role check in `server`, add a row here.

## Roles

Defined per [Membership](resources.md#membership).

| Role | Granted by | Summary |
|------|------------|---------|
| `owner` | Pod creation | Full control over the pod, its members, and its content |
| `member` | Invite accepted | Read/write tasks, invite further users |
| `viewer` | Invite accepted | Read-only access to the pod |

## Access tiers

Authorisation is layered. Each tier below must pass before the next is checked.

1. **Authentication** — every non-public route requires a valid bearer token tied to a known user. One route additionally accepts an API key — see [Authentication methods](#authentication-methods).
2. **Pod access** — routes scoped to a pod (`/beta/pods/{podId}/...`) additionally require the caller to hold an *accepted* membership for that pod. Non-members and users with only pending invites receive `403`. The only authenticated route a pending invitee can call usefully is `GET /beta/invites`.
3. **Role check** — within a pod-scoped route, the role tables below distinguish `owner`, `member`, and `viewer`. These run after the pod-access check has already established that the caller is some kind of member.

## Invites & Membership

Flow: [Invites & Membership](flows/invites.md).

| Action | owner | member | viewer | Notes |
|--------|:-----:|:------:|:------:|-------|
| Create invite | ✓ | ✓ | | Target role limited to `member` or `viewer` |
| Cancel pending invite | ✓ (any) | ✓ (own) | | Inviter must still be an accepted member |
| Remove member | ✓ | | | Owner cannot remove themselves |
| List pod members | ✓ | ✓ | ✓ | |
| List pod invites | ✓ | ✓ | ✓ | |
| Accept / reject own invite | — | — | — | Invitee acts before they hold a role |

## API Keys

Flow: [API Keys](flows/api-keys.md).

All roles can mint and list keys: API-key requests reach only a read-only endpoint (see [Authentication methods](#authentication-methods)), so a viewer minting a key delegates the read access they already have to a device rather than expanding their capability.

| Action | owner | member | viewer | Notes |
|--------|:-----:|:------:|:------:|-------|
| Create API key | ✓ | ✓ | ✓ | |
| List pod API keys | ✓ | ✓ | ✓ | |
| Delete API key | ✓ (own) | ✓ (own) | ✓ (own) | Only the creator may delete; non-creator deletions receive `403` |

## Authentication methods

The role matrix above governs **bearer-authenticated** requests (a logged-in user accessing the API on their own behalf). API-key-authenticated requests — used by the physical controller, see [Authentication](authentication.md) — have a much narrower surface area. Exactly one endpoint accepts an API key:

| Endpoint                           | Method |
|------------------------------------|--------|
| `/beta/pods/{podId}/device/status` | `GET`  |

All other routes require bearer authentication, including API-key management itself: a controller cannot self-manage its keys or change its own slot assignments.

## Conventions for this page

- One section per resource or feature area; one row per server-side authorisation check.
- "✓" means allowed; blank means denied. Use "✓ (qualifier)" when the rule is narrower than the role itself (e.g. "own", "any").
- If an action has its own per-route preconditions beyond the role (e.g. "owner cannot remove themselves"), put them in the **Notes** column rather than splitting rows.
- Link the section header — or a dedicated row's **Notes** — to the flow doc that documents the state transitions and error codes.
