# la-truffe-ai-pilots

Concise Svelte landing page for 3 AI pilot concepts tailored to La Truffe Sauvage.

The `reservation-yield` route now includes a day-of MVP reservation board with:
- v1 data model spec (tables, reservations, events)
- 3 toggleable board variants (`v1` control board, `v2` floor-first, `v3` action queue)
- shift mode for each variant (larger touch targets and simplified actions)
- table assignment suggestions and one-tap best assignment
- quick status actions (arrived, seated, late, no-show, cancelled)
- walk-in capture
- nightly agent call list and load curve preview

## Local dev

```bash
npm install
npm run dev
```

## Build

```bash
npm run build
npm run preview
```

## Deploy to GitHub Pages

1. Push this repository to GitHub as `camerondurham/la-truffe-ai-pilots`.
2. In GitHub repo settings, set Pages source to **GitHub Actions**.
3. The included workflow in `.github/workflows/deploy.yml` will build and deploy on every push to `main`.

If the repository name changes, update `base` in `vite.config.js`.
