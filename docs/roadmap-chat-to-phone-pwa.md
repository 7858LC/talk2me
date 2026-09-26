# Roadmap: Claude Chat → Claude Code → Live PWA on Phone

Repeatable process for turning a build prompt written in Claude chat into a
working, installable app on your phone. Hosting/deploy steps reflect a
pipeline already proven on a prior build (GitHub Pages via GitHub Actions),
not a hypothetical one.

**Scope:** this is the pattern for a *static, client-only* app — everything
runs in the browser, data stays on the device. If the app needs a server
(an LLM or speech API call, accounts, shared data, anything with a secret
key), the hosting stage changes: a static host can't hold an API key, since
anything shipped to the browser is readable by anyone who opens the page.
Settle that in Stage 1 before writing the build prompt.

## Stage 1 — Design in Claude chat

1. Road to Damascus questions first: what you want when it's done, what you
   can't afford to redo, what's good enough for today.
2. Lock the data schema and screen list — the expensive-to-redo part.
3. Build prompt delivered as a single markdown file (copies cleanly).
4. **State these in the prompt, not later** — each one changes Stage 4:
   - Client-only, or does it need a server / API key? (see Scope above)
   - Offline launch required? (needs a service worker, e.g.
     `vite-plugin-pwa`; without one, saved data survives offline but the
     app itself won't open without a signal)
   - Hosting target (default: GitHub Pages) and repo visibility (see
     step 14)

## Stage 2 — Hand off to Claude Code

5. Start a Claude Code session on the `talk2me` repo. (Cloud sessions have
   no `gh` CLI, so repos get created on github.com first — already done
   here.)
6. Paste the build prompt file as the first message.
7. Claude Code should, in order:
   - Scaffold (default: React + TypeScript + Vite unless the prompt says
     otherwise)
   - Build the data layer first, with unit tests on the logic
   - **Confirm the schema back in plain language before writing UI** —
     your drift checkpoint
   - Build screens in the prompt's order
8. Verification: in a **cloud** session you can't click through `npm run
   dev` yourself — ask for a Playwright smoke test of the full
   click-through. In a **local** CLI session, run `npm run dev` and click
   through yourself.

## Stage 3 — Git

9. Cloud sessions push to a `claude/...` branch, not `main`. Nothing
   deploys until that branch is merged to `main` via a PR. Budget the merge
   as a real step.

## Stage 4 — Hosting (GitHub Pages via Actions)

10. Add `.github/workflows/deploy.yml`: `npm ci` → `npm test` →
    `npm run build` → deploy to Pages, on push to `main`. Tests gate the
    deploy — a failing test means no deploy, which is the point.
11. Set Vite `base` to `'/talk2me/'` for the Pages build only (key it off
    an env var such as `GITHUB_PAGES` so local dev stays at `/`). Manifest
    `start_url`, `scope`, and icon paths must use the same subpath — a
    mismatch silently breaks install.
12. Repo Settings → Pages → Source = **GitHub Actions** (one-time, manual).
13. Result: `https://7858lc.github.io/talk2me/`. HTTPS is what makes it
    installable.
14. **Visibility blocker:** `talk2me` is currently **private**. On a
    personal Free plan, Pages only publishes from public repos. Pages on
    a private repo needs a paid plan (Pro), and even then the site itself
    is public — only Enterprise Cloud can make a Pages site private.
    Options: make the repo public (fine for a client-only app with no
    secrets or personal data committed), upgrade, or pick another host
    (Vercel/Netlify/Cloudflare Pages all deploy from private repos on
    free tiers).

## Stage 5 — Install on your phone

15. Open the live URL on the phone.
16. iOS Safari: Share → Add to Home Screen (manual; needs an
    `apple-touch-icon` link and `apple-mobile-web-app-*` meta tags).
    Android Chrome: install prompt or menu → Install app, driven by a
    valid manifest (name, icons 192 + 512, `display: standalone`).
17. Confirm standalone launch (no browser chrome) and the correct icon.
    Test offline launch only if a service worker was built.
18. Data stored in the browser (IndexedDB/localStorage) lives only on that
    phone. Clearing site data wipes it, and it does not sync between
    devices. Build an export/backup into the app if the data matters.

## Stage 6 — Ongoing changes

19. Ask Claude Code for the change → it commits and pushes to its branch →
    **merge the PR to `main`** → Actions tests and redeploys. No manual
    upload.
20. Any change to the on-device data schema needs a versioned migration
    (e.g. a Dexie version bump), or existing data on the phone can break.
    Call it out in the change request.
21. For a schema-drift / scope-creep review, paste the diff into chat
    before merging the PR.
