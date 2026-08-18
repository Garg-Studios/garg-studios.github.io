# Agent Guide — Penguin Development Website

Detailed reference for working in this repo. `CLAUDE.md` has the quick-glance version; this file has the reasoning behind it.

## What this is

A static marketing/portfolio site for Penguin Development, served directly from GitHub Pages at [penguindevelopment-official.github.io](https://penguindevelopment-official.github.io). Plain HTML/CSS/JS — no framework, no bundler, no `package.json`, no build step. What's in the repo is byte-for-byte what gets served.

## Structure

- `index.html` — homepage, links out to each project page
- `Pages/*.html` — one file per project/legal page (ToS, privacy, licensing, per-project pages like `exchangebot.html`, update logs, etc.)
- `Static/Style/style.css` — the **only** stylesheet, shared by every page
- `Static/JS/main.js` — tiny helper (currently just a clipboard-copy function used by the copy-URL button)
- `Static/Assets/` — fonts and images

Every HTML page follows the same skeleton: `<meta charset>`, `<meta name="viewport">`, `<title>`, favicon link, stylesheet link, ionicons CDN scripts, then a `<header>` (home icon + page title + logo), a `<main>`, and a shared-style `<footer>`.

## Deployment

Pushing to `main` deploys automatically — GitHub Pages rebuilds in well under a minute (confirm with `gh run list --limit 3`, look for a fresh `pages-build-deployment` run). There's no Jekyll processing happening in practice (no `_config.yml` / `.nojekyll`, verified the live HTML is identical to the repo copy) — files are served as-is.

The repo's canonical location is `penguindevelopment-official/penguindevelopment-official.github.io` (transferred from the `Garg-Studios` org, which no longer exists). Unlike an in-org rename, an ownership transfer to a different org does **not** redirect the old GitHub Pages domain — `garg-studios.github.io` now 404s outright, so any references to it need to be updated by hand, not left to a redirect. Individual repo paths (e.g. `github.com/<old-org>/<repo>`) do still redirect via GitHub's rename tracking, but the `<owner>.github.io` Pages hostname does not, since it's tied to the org identity itself. The local `origin` remote was updated to `https://github.com/penguindevelopment-official/penguindevelopment-official.github.io.git` after the transfer.

Branch protection on `main` requires PRs, but the repo owner has bypass rights, so direct pushes succeed — `git push` will print a "Bypassed rule violations" notice, which is expected and not an error. Still, treat `main` as production: confirm with the user before pushing, since every push is a live deploy.

**Always verify a deploy landed by fetching the live file, not just by pushing:**
```bash
curl -s "https://penguindevelopment-official.github.io/Static/Style/style.css" | grep -A5 ".some-rule {"
gh run list --limit 3
```
This has repeatedly caught the real culprit when a fix "didn't work" — it almost always had, and the browser (especially mobile Safari, but Chrome too) was just serving a stale cached copy of the CSS. Before concluding a pushed fix is broken, rule out client-side caching first: confirm the live file via `curl`, and if it's correct, tell the user to hard-refresh or use a private/incognito tab rather than re-diagnosing a CSS bug that isn't there.

## CSS conventions — read before touching `style.css`

The stylesheet is one file, and it deliberately has two zones:

1. **Everything above `/* MOBILE ONLY CSS */`** is the desktop baseline. These rules are what render on any screen wider than 768px, and several of them are intentionally quirky (e.g. `.projectcontainer` uses `width: 100vh`, sizing the card off the viewport *height* — that's an original-author choice that happens to look right on typical desktop windows; it's not a bug to "fix" on sight).
2. **The `@media (max-width: 768px) { ... }` block at the bottom** holds every mobile-specific override. This split exists on purpose, after an earlier round of edits directly to the base rules kept leaking into desktop and regressing it. **Never edit a base rule to fix a mobile issue — add or adjust a rule inside the media query instead.** Conversely, desktop-only requests should never touch anything inside the media query.

When adding a new mobile override, prefer a selector-specific fix (e.g. target the exact class an image or element uses) over changing a widely-shared class like `.containsimg`, which is reused across the header logo, project-name rows, and the copy-URL button — a mobile override on the shared class will hit all of them.

Avoid relying on `align-items: stretch` for critical layout sizing — it's been unreliable in Safari for flex items with intrinsic content. Prefer giving elements an explicit `width: 100%` (or similar definite size) over depending on stretch behavior.

## Testing — no framework, verify visually

There's no test suite. To check a change before pushing:

```bash
python3 -m http.server 8942   # from repo root
```
Then use the browser preview tool at both a mobile viewport (375×812) and a desktop-sized one (e.g. 1280×900), and spot-check element dimensions via `javascript_tool` (`getBoundingClientRect()`, `getComputedStyle()`) rather than eyeballing alone — this caught several regressions (unequal card widths, horizontal overflow, a shrunk desktop logo) that weren't obvious from a screenshot.

**Known limitation:** there's no way to test real Safari from this environment (iOS Simulator needs a full Xcode install, and the browser automation tools here are Chromium-based only). When a bug is reported as Safari-specific, reason carefully from known WebKit flexbox/CSS quirks and say explicitly that the fix is unverified in real Safari — don't claim confidence you don't have. Ask the user for a screenshot from the actual device when precision matters.

## Git conventions

- Work happens directly on `main` — this project doesn't use feature branches or PRs in practice, despite the branch-protection rule.
- Confirm with the user before pushing; every push is a live production deploy with no staging environment.
- Commit messages here are short, plain-English, present/past-tense descriptions of the visible change (see recent `git log` for tone).
