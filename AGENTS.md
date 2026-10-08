# AGENTS.md – slidev-addon-autofit

## Project overview

A [Slidev](https://sli.dev) addon that automatically scales a slide down until its content fits, similar to PowerPoint's "shrink text on overflow". It drives Slidev's own per-slide zoom (`--slidev-slide-zoom-scale`), so it works with any theme and layout.

- **npm package:** `slidev-addon-autofit`
- **GitHub:** `https://github.com/L-C-P/slidev-addon-autofit`
- **License:** MIT
- **Author:** Denis Sowa

---

## Project structure

| Path | Description |
|---|---|
| `slide-top.vue` | Per-slide layer: measures overflow, searches the fitting scale, sets the zoom variable |
| `slides.md` | Demo/preview slides for this addon |
| `.github/workflows/publish.yml` | CI/CD workflow – publishes to npm on tag push |

---

## Key concepts

- Slidev renders `slide-top.vue` of every addon inside each slide (presentation, presenter view, overview, export).
- Fitting runs synchronously inside `ResizeObserver` / `MutationObserver` callbacks – after layout, before paint. The slide must never visibly shrink after navigation.
- The slide component is loaded asynchronously: observe the whole slide page, not only the layout element.
- Slidev's slide transitions use `transition: all`. The class `slidev-autofit` is set permanently on the slide page and limits transitions to `translate`, `transform`, `opacity`, `rotate`, `filter`, `clip-path`. Never toggle it while measuring – every forced style recalculation would start a transition of the scale.
- A manual `zoom:` in the frontmatter takes precedence; `autofit: false` disables the addon.

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
- Owner: `L-C-P`, Repository: `slidev-addon-autofit`, Workflow: `publish.yml`
- The workflow requires `permissions.id-token: write` for OIDC authentication.

---

## Development

```bash
npm install       # install dependencies
npx slidev        # start Slidev dev server with the demo slides
```

### Important notes
- Always run `npm version patch/minor/major` from a **clean** working directory.
  If the directory is dirty, use `npm version patch --no-git-tag-version` to bump only `package.json`/`package-lock.json`, then commit manually and tag separately.
- Do not commit `node_modules`.
- All variables, comments, and commit messages must be in **English**.
- German text (e.g. in slides) must use UTF-8 encoded Umlauts.
