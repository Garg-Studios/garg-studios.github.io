# CLAUDE.md

Static HTML/CSS/JS site for Penguin Development, deployed via GitHub Pages at [penguindevelopment-official.github.io](https://penguindevelopment-official.github.io). No framework, no build step, no package manager — what's committed is what's served.

**Read [AGENT_GUIDE.md](AGENT_GUIDE.md) before making non-trivial changes.** It has the full reasoning; this file is just the fast-reference version.

## Fast facts

- One shared stylesheet: `Static/Style/style.css`. Desktop rules live above `/* MOBILE ONLY CSS */`; everything mobile-specific belongs inside the `@media (max-width: 768px)` block at the bottom. Don't cross the streams — a mobile fix goes in the media query, never in a base rule, and vice versa.
- Pushing to `main` deploys live immediately (direct pushes are allowed via branch-protection bypass). Confirm with the user before pushing.
- After pushing, verify the live file with `curl` before trusting a screenshot or user report that a fix "didn't work" — GitHub Pages updates fast, but browsers (mobile Safari especially) cache CSS aggressively and often show stale content.
- Every stylesheet `<link>` uses a cache-busting `?v=2` query string. **Bump it in all 15 HTML files whenever `style.css` changes** — otherwise a visitor can get fresh HTML with stale CSS, which once caused a real production bug (unstyled `<img>` rendering at its native 1024px size).
- No test suite. Verify changes by serving locally (`python3 -m http.server`) and checking both a mobile (375×812) and desktop (~1280×900) viewport via the browser tool, measuring actual element dimensions rather than eyeballing.
- No real Safari testing available here (no full Xcode install). Say so explicitly when a fix targets a Safari-only bug.

See [AGENT_GUIDE.md](AGENT_GUIDE.md) for repo structure, deployment details, and the CSS conventions in full.
