# AGENTS.md – slidev-addon-second-screen

## Project overview

A [Slidev](https://sli.dev) addon that adds a button to the presenter view for opening the presentation on a second screen in fullscreen, similar to PowerPoint's presenter view. Uses the Window Management API to detect and target the correct display.

- **npm package:** `slidev-addon-second-screen`
- **GitHub:** `https://github.com/L-C-P/slidev-addon-second-screen`
- **License:** MIT
- **Author:** Denis Sowa

---

## Project structure

| Path | Description |
|---|---|
| `custom-nav-controls.vue` | Registers the second screen button in Slidev's presenter nav bar |
| `components/SecondScreenControls.vue` | Main Vue component – button logic, Window Management API, fullscreen handling |
| `slides.md` | Demo/preview slides for this addon |
| `.github/workflows/publish.yml` | CI/CD workflow – publishes to npm on tag push |

---

## Key concepts

- Requires **Chrome 100+ or Edge 100+** (Window Management API / `getScreenDetails()`).
- Requires a **Secure Context** (HTTPS or `localhost`).
- Opens the presentation on the screen where the presenter is **NOT** running.
- Slidev's built-in navigation sync keeps both windows in step automatically.
- `AUDIENCE=bypass` is not relevant here (no preparser involved).

---

## Publish workflow (CI/CD)

Publishing to npm is fully automated via GitHub Actions using **Trusted Publishing (OIDC)** – no npm token or secret required.

### Trigger
A push of a version tag (`v*`) triggers the workflow in `.github/workflows/publish.yml`.

### Steps to release a new version

1. Ensure the working directory is clean (`git status`)
2. Bump the version and create a tag:
   ```bash
   npm version patch   # or minor / major
   ```
3. Push the commit and tag:
   ```bash
   git push --follow-tags
   ```
4. GitHub Actions picks up the tag and runs `npm publish --provenance --access public` automatically.

### Trusted Publishing setup (must be configured on npmjs.com)
- Configure on npmjs.com under the package settings → *Trusted Publishers*
- Owner: `L-C-P`, Repository: `slidev-addon-second-screen`, Workflow: `publish.yml`
- The workflow requires `permissions.id-token: write` for OIDC authentication.

---

## Development

```bash
npm install       # install dependencies
npm run dev       # start Slidev dev server with demo slides (if script exists)
```

### Important notes
- Always run `npm version patch/minor/major` from a **clean** working directory.
  If the directory is dirty, use `npm version patch --no-git-tag-version` to bump only `package.json`/`package-lock.json`, then commit manually and tag separately.
- Do not commit `node_modules`.
- All variables, comments, and commit messages must be in **English**.
- German text (e.g. in slides) must use UTF-8 encoded Umlauts.
