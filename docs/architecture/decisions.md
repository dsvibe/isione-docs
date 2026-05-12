# Interface Decisions

Technical decisions affecting the API and cross-component interfaces.

## OIDC Provider Neutrality

The API treats the `Authorization: Bearer` token as opaque OIDC and does not read provider-specific claims. [Authentik](https://auth.dsvibe.io) is the current deployed provider; any OIDC-conformant provider could be swapped in without changing the API contract.

This preserves the option to migrate identity providers without coordinated changes across web, mobile, and the physical controller.

## Durations in Microseconds

Duration values (e.g. `pendingAfter`, `overdueAfter`) are represented as integers in microseconds.

This aligns with PostgreSQL's `interval` type, which stores durations with microsecond precision.

| Duration | Microseconds |
|----------|--------------|
| 1 hour | 3,600,000,000 |
| 1 day | 86,400,000,000 |
| 1 week | 604,800,000,000 |
