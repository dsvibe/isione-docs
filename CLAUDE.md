# IsiOne Documentation

Developer documentation for the IsiOne project. This repo documents interfaces, APIs, and flows between components—not implementation details. Each component's internal documentation lives in its own repo.

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
