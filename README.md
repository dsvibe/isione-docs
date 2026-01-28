# IsiOne Documentation

Developer documentation for the IsiOne project, hosted at [dsvibe.github.io/isione-docs](https://dsvibe.github.io/isione-docs/).

## Local Development

```bash
docker compose run --rm --service-ports docs
```

Then open http://127.0.0.1:8000

## Deployment

The site deploys automatically to GitHub Pages when changes are pushed to `main`.

To enable deployment, go to repo Settings → Pages → Source and select **GitHub Actions**.

## Structure

```
index.md              # Homepage
architecture/         # System design, data flow, component interactions
api/                  # API reference documentation (planned)
server/               # Server-specific docs (planned)
web/                  # Web frontend docs (planned)
android/              # Android app docs (planned)
controller/           # ESPHome controller docs (planned)
```
