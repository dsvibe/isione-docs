# Authentication

The IsiOne API recognises two authentication paths. Users authenticate via OIDC; physical controller devices authenticate via per-pod API keys. The two paths converge on a single route — see [Where each is accepted](#where-each-is-accepted).

## Two auth paths

|             | OIDC (user)                                | API key (device)                                              |
|-------------|--------------------------------------------|---------------------------------------------------------------|
| Header      | `Authorization: Bearer <token>`            | `X-API-Key: <key>`                                            |
| Issued by   | [Authentik](https://auth.dsvibe.io)        | The IsiOne API, via the pod-settings UI                       |
| Identifies  | A user                                     | A pod                                                         |
| Pod scope   | URL `{podId}` + accepted-membership check  | Implied by the key (per-pod)                                  |
| Lifetime    | Token expiry; refresh via Authentik        | Until deleted, or until the creator loses accepted membership |
| Accepted on | All non-public routes                      | One route — see [below](#where-each-is-accepted)              |

## OIDC (user authentication)

The bearer token is issued by [Authentik](https://auth.dsvibe.io) and identifies the user making the request. The API treats the token as opaque OIDC and does not read provider-specific claims; the provider could be swapped without changing the API contract — see [Decisions: OIDC Provider Neutrality](decisions.md#oidc-provider-neutrality).

Pod-scoped routes (`/beta/pods/{podId}/...`) also require the caller to hold an accepted membership for that pod — see [Permissions: Access tiers](permissions.md#access-tiers).

The OpenID issuer URL and other service endpoints are listed in the [Services table](../index.md#services).

## API keys (device authentication)

An API key is a long-lived, per-pod credential. Each key is bound to the user who created it: the key authenticates only while that user holds an accepted membership for the pod. If the creator's membership is cleared — an owner removes them, or they leave — every key they created stops authenticating, without an explicit `DELETE`. This is a deliberate revocation primitive.

If the creator is later re-invited and accepts, their previously-created keys begin authenticating again. To make revocation permanent, delete the keys explicitly. The [API Keys flow](flows/api-keys.md) covers operational guidance: rotation, removing a single controller, and recovery.

### Key format

- Plaintext keys are 64 lowercase hex characters.
- Sent on each request in the `X-API-Key` header.

### One-shot plaintext

The full plaintext key is returned exactly once, in the `201` response to `POST /beta/pods/{podId}/api-keys`. The list endpoint omits the plaintext entirely. If a device loses its key, the user must create a new one and delete the old.

### lastUsed

The API Key resource exposes a `lastUsed` field (nullable ISO timestamp). The server sets it when the key successfully authenticates a request.

## Where each is accepted

| Route                                            | Bearer | API key |
|--------------------------------------------------|:------:|:-------:|
| `GET /beta/pods/{podId}/device/status`           | ✓      | ✓       |
| All other routes (incl. all API-key management)  | ✓      |         |

A physical controller cannot self-manage its keys, change its own slot assignments, or read any pod data outside `device/status`. All API-key management is bearer-only.

The same matrix is referenced from the role-table side in [Permissions: Authentication methods](permissions.md#authentication-methods).

## API-key failure modes

When `X-API-Key` is present, the server responds:

| Status | Body                       | Cause                                                                                  |
|--------|----------------------------|----------------------------------------------------------------------------------------|
| `401`  | `"invalid API key format"` | Header is not exactly 64 lowercase hex characters                                       |
| `401`  | `"invalid API key"`        | Well-formed key, but unknown — or the creator no longer holds an accepted pod membership |
| `403`  | —                          | Well-formed and known, but the key was minted for a different pod than the URL `{podId}` |
| `400`  | —                          | URL `{podId}` is not a valid UUID                                                       |

For a frontend or firmware client, the meaningful split is between the two `401` cases: `"invalid API key format"` indicates a bug in the client (wrong shape); `"invalid API key"` indicates the key has been deleted, rotated, or the creator has lost access — the device needs re-pairing.

If the `X-API-Key` header is absent, the request is treated as bearer-authenticated; the absence of the header is not an API-key failure.
