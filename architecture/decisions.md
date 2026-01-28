# Interface Decisions

Technical decisions affecting the API and cross-component interfaces.

## Durations in Microseconds

Duration values (e.g. `pendingAfter`, `overdueAfter`) are represented as integers in microseconds.

This aligns with PostgreSQL's `interval` type, which stores durations with microsecond precision.

| Duration | Microseconds |
|----------|--------------|
| 1 hour | 3,600,000,000 |
| 1 day | 86,400,000,000 |
| 1 week | 604,800,000,000 |
