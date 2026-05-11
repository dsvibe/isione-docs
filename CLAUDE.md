# IsiOne Documentation

Developer documentation for the IsiOne project. This repo documents interfaces, APIs, and flows between components—not implementation details. Each component's internal documentation lives in its own repo.

## Scope: cross-component, not implementation

This is the **hardest** rule to apply because the source-of-truth for any flow lives in one component's repo, and it's tempting to copy specifics across. Don't. A reader of this repo should understand how the components fit together without ever needing to know how any one of them is built.

**Belongs here**

- HTTP endpoints, methods, request/response shapes, status codes
- Cross-component state machines and how state is observable through the API
- Permission rules expressed in role/action terms
- Conceptual data model (resource names, public field names, relationships)
- Sequence diagrams showing interactions between components
- Cross-cutting decisions (e.g. duration units, ID formats)

**Does NOT belong here**

- Source file paths, function names, class names, package names from any component repo (`handlers/invites.go`, `UpsertUser`, `RequirePodAccess`, `app/api/invites.ts`, etc.)
- Database column names or schema details (`accepted_at`, `user_id`, `NOT NULL`) — use the public API field names (`acceptedAt`) and describe behaviour, not storage
- Middleware/framework names, library choices, ORM specifics
- Build tooling, test layouts, lint config — anything a reader of *one* component would care about but a reader of *another* wouldn't
- "Server-side reference" sections that link into another repo's source tree

**Test**: if the server team rewrote their Go service in Rust tomorrow, or the web team switched from React Router to TanStack, would this page need to change? If yes for reasons other than "the public API changed", it's leaking implementation.

**When in doubt**, prefer describing observable behaviour ("marks the invite as rejected", "the membership is cleared, allowing re-invite") over describing how the component achieves it ("sets `rejected_at = now()`", "deletes the row").

## Project Components

| Component | Description | Repo |
|-----------|-------------|------|
| server | Backend API | [isione-server](https://github.com/dsvibe/isione-server) |
| web | Web frontend | [isione-web](https://github.com/dsvibe/isione-web) |
| android | Android app | [isione-android](https://github.com/dsvibe/isione-android) |
| ios | iOS app (planned) | - |
| controller | IoT status LEDs | [isione-controller](https://github.com/dsvibe/isione-controller) |
| infrastructure | Deployment & infra (dsvibe-wide) | [infrastructure](https://github.com/dsvibe/infrastructure) |
| website | dsvibe company website | [website](https://github.com/dsvibe/website) |

These repos are private and not accessible via GitHub. Assume they are available locally:

```
dsvibe/
├── infrastructure/
├── website/
└── isione/
    ├── android/
    ├── controller/
    ├── docs/          # (this repo)
    ├── ios/
    ├── server/
    └── web/
```

## Documentation Structure

```
architecture/     # System design, data flow, component interactions
style-guide.md    # Cross-platform design: icons, task fonts
api/              # API reference documentation
server/           # Server-specific docs
web/              # Web frontend docs
android/          # Android app docs
ios/              # iOS app docs
controller/       # ESPHome controller docs
hardware/         # Physical components: enclosures, labels, NFC tags
```
