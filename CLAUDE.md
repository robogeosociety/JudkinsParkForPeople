# Claude Context for JudkinsParkForPeople

**CRITICAL: All deployments must follow the deployment protocol below and the issue protocol.**

## Deployment

Two workflows:

1. **Staging** (`deploy-staging.yaml`): Auto-deploys on PRs to `main`. Takes a Playwright screenshot and posts the staging URL + screenshot artifact as a PR comment. Warns when one PR's staging overwrites another's (single shared `/staging/` directory). Verify at `https://judkinsparkforpeople.org/staging/`.
2. **Production** (`deploy-spa.yaml`): Auto-deploys on merge to `main`. Verify at `https://judkinsparkforpeople.org/`.

## Project Structure

```
JudkinsParkForPeople/
├── .github/workflows/       # CI/CD (deploy-spa.yaml, deploy-staging.yaml)
├── app/                     # Vite React SPA
│   ├── src/
│   │   ├── App.jsx          # Main scrollytelling component + CHAPTERS data
│   │   ├── App.test.jsx     # Smoke tests
│   │   ├── setupTests.js    # Vitest setup, mocks mapbox-gl & react-map-gl
│   │   ├── index.css        # Tailwind v4 entry (@import "tailwindcss")
│   │   └── main.jsx         # React entry point
│   ├── public/favicon.svg
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   └── eslint.config.js
├── PROJECT.md               # Original technical specification
└── GEMINI.md                # Project context document
```

## Tech Stack

- **React 19** + **Vite 7** + **Tailwind CSS v4** (`@tailwindcss/vite`)
- **Mapbox** via `react-map-gl/mapbox` (v8 subpath export) + `mapbox-gl` v3
- **Scrollytelling**: `react-scrollama` (Scrollama + Step components)
- **Animation**: `framer-motion`
- **Icons**: `lucide-react`
- **Content**: `react-markdown` for chapter descriptions
- **Testing**: Vitest + Testing Library

## Mapbox Token Management

- Token: `VITE_MAPBOX_ACCESS_TOKEN` in `app/.env` (gitignored)
- CI/CD: uses `secrets.MAPBOX_ACCESS_TOKEN` GitHub secret
- Local setup: copy token from `~/.mapbox/credentials` into `app/.env`

## Development Protocol

When performing development or deployment tasks:

1. **Branch isolation**: Develop on a descriptive feature branch. Never commit directly to `main`.
2. **Verify domain coexistence**: All workflow changes must preserve `keep_files: true` to prevent the production root and `staging/` directory from overwriting each other on `gh-pages`.
3. **Merging PRs**: Do not merge PRs unless explicitly instructed by the user (e.g. "merge the PR"). When instructed, use `gh pr merge --merge`.
3. **Run tests before opening PRs**: `cd app && npm test && npm run lint`

## Issue Lifecycle

### Phase 1: Exploration
- Use Grep/Glob to locate relevant components before proposing changes.
- The narrative chapter data lives in `app/src/App.jsx` as the `CHAPTERS` array.

### Phase 2: Implementation & Verification
1. Implement on a feature branch.
2. Run `npm test` and `npm run lint` from `app/`.
3. Fix any failures before opening a PR.

### Phase 3: Pull Request & Staging
1. Open a PR to `main`, using `Fixes #N` or `Closes #N` in the PR body.
2. `deploy-staging.yaml` triggers automatically, deploys to `/staging/`, takes a screenshot, and posts the URL as a PR comment.
3. Respond to review feedback with new commits. Only merge when explicitly instructed.

### Phase 4: Deployment & Closure
1. After merge to `main`, `deploy-spa.yaml` deploys automatically.
2. Verify at `https://judkinsparkforpeople.org/`.
3. Only close the issue after confirming production deployment succeeded.

## Scrollytelling Architecture

The app uses a **fixed map + scrollable narrative** pattern:
- `Map` (react-map-gl) is `position: fixed` behind everything
- `Scrollama` wraps `Step` components, each being a `min-h-screen` section
- `onStepEnter` triggers `mapRef.current.flyTo()` to animate the map
- Chapter cards use `framer-motion` for entry animation
- `CHAPTERS` array in `App.jsx` defines all narrative stages and map states

## Styling

- "Transit-Modern" palette: deep blue `#1e3a8a`, safety green `#16a34a`, orange `#ea580c`, purple `#7c3aed`
- Glassmorphism cards: `bg-white/85 backdrop-blur-md`
- Tailwind v4 — no `tailwind.config.js` needed, just `@import "tailwindcss"` in CSS
## Pull requests — the "newspaper" framework

PR descriptions follow the **newspaper / information-pyramid** format: one self-contained
front page (kicker → headline → dek → masthead → why → what → mermaid flow → screens →
verification → risk) that reads top-to-bottom on an iPad-mini portrait display (1–2 pages;
up to 4 for very complex *code* changes). Rebuild from the **full** diff, never append.
Full rules: <https://github.com/tommyroar/.github/blob/main/PR_FRAMEWORK.md>. CI validates
the body via the `pr-newspaper` workflow (the reusable gate in `tommyroar/pr-newspaper`).
