# CLAUDE.md

Guidance for Claude Code working in this repo.

## What this is

`mks.sh` — a single-file, static personal portfolio styled as a terminal
session. Deployed as a GitHub Page at **mks-01.github.io** (repo
`MKS-01/about-me`). Everything lives in **`index.html`**: inline `<style>`,
inline SVG, and one inline `<script>` at the bottom. There is no build step,
no framework, no dependencies — open the file and it runs.

## Read the skill first

Before ANY visual, copy, layout, or animation change, invoke the
**`portfolio-conventions`** skill (`.claude/skills/portfolio-conventions/`).
It is the source of truth for the design tokens, the terminal conceit, the
scroll-snap deck, the fixed CSS MacBook, the tmux status-bar nav, the mobile
tiers, and — critically — a long list of **already-rejected ideas** with the
owner's own words. Re-proposing a loser wastes everyone's time; the skill
exists so you don't.

## Verifying changes (no dev server)

Render with headless Chrome and READ the screenshot back — don't eyeball the
diff. The skill documents the exact invocation and the gotchas (fragment
URLs render blank; headless clamps width to 500px min; `position: fixed` mac
sits at the bottom of a tall stacked render). For mobile checks, use a test
copy with `body { width: 390px }` and probe layout via an injected script +
`--dump-dom`, because the real viewport can't be forced below 500px.

Note: this headless environment refuses programmatic scrolling (even a plain
`window.scrollTo` reports 0), so scroll-driven behavior and tap navigation
can't be exercised here — verify those on a real device after deploy.

## Mobile is the sharp edge

Most bugs in this project have been mobile-specific, and iOS Safari behaves
differently from Android:

- iOS Safari **rarely collapses its URL bar** under scroll-snap, so iPhones
  sit at ~620–760px viewport height permanently. Height-threshold rules that
  assume the bar collapses will misfire on iOS while looking fine on Android.
- `env(safe-area-inset-*)` only works with `<meta viewport-fit=cover>`.
- Hash-anchor nav fights `scroll-snap-type: mandatory` on iOS — nav is
  JS-driven (see the skill).

Always test the short-viewport and safe-area cases, not just a tall phone.

## Workflow norms

- **Commit directly to `main`** (this repo's pattern) only when the owner
  says so; then `git push` — the live site is GitHub Pages off `main`, so
  nothing reaches the owner's phone until pushed.
- Commit message co-author trailer:
  `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>`
- For subjective/creative choices (copy, layout style, colors) present 2–4
  concrete options via AskUserQuestion — this owner consistently prefers
  picking from real options over open-ended prose.
- When a change encodes a hard-won lesson or a rejected direction, record it
  in the `portfolio-conventions` skill so the next session doesn't repeat it.
