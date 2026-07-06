---
name: portfolio-conventions
description: Design system, content voice, and animation conventions for MKS's mks.sh terminal-style portfolio (index.html). Use before making ANY visual or copy change to this page — new sections, rewording lines, adjusting colors/spacing, or touching the entrance animation.
---

# mks.sh portfolio conventions

Single-file static page (`index.html`) styled as a terminal session. Everything —
header, whoami, project list, now-reading, footer — reads like one continuous
shell session, top to bottom. Every design and copy decision should reinforce
that illusion, not break it.

## Design tokens (`:root`)

```
--bg: #0d1117        page background
--text: #c9d1d9       primary text (bold lines, links' hover)
--text-dim: #8b949e   secondary text (most body copy)
--text-faint: #484f58 separators (·, →), faint icons
--accent: #58a6ff     links, project names, leet tokens
--green: #3fb950      the "$ " prompt glyph — ONLY use for prompts
--ease-out: cubic-bezier(0.23, 1, 0.32, 1)   strong ease-out, used everywhere
```

Font: Fira Code, 14px, line-height 1.7. Don't introduce a second font or
break monospace anywhere — the terminal conceit depends on it.

## Layout rules

- Page is vertically **and** horizontally centered (`body` flex + `align-items:
  center`, fixed `padding: 3rem 1.5rem` on `main`). Do not go back to
  top-anchored layout with huge bottom whitespace — that was explicitly
  rejected ("looks like hanging on the top").
- `main { max-width: 720px }` — keep line lengths terminal-narrow.
- Every section is a `<p class="prompt">` (green `$ command`) followed by
  `<div class="output">` / a result paragraph. New content should follow this
  exact pattern, not a departure into card/grid layouts.
- **No horizontal divider lines between sections.** A `border-top` on the
  footer was tried and explicitly removed — a real shell never draws a rule
  between commands; blank space alone separates blocks. Don't reintroduce
  borders.
- Footer is **left-aligned**, not centered — it's a continuation of the
  terminal stream, not a separate "site footer" block. Centering it was tried
  and rejected for looking "separate."

## Entrance animation ("terminal print")

Every visible line has class `in` plus an inline `style="--i:N"` giving its
print order (0-indexed, sequential across the whole page — header is 0,
last footer line is highest). CSS:

```css
.in {
  opacity: 0;
  animation: printIn 320ms var(--ease-out) both;
  animation-delay: calc(var(--i, 0) * 55ms);
}
@keyframes printIn {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}
@media (prefers-reduced-motion: reduce) {
  .in { animation-duration: 1ms; transform: none; }
}
```

This follows the `emil-design-eng` / `animation-vocabulary` skills in
`.agents/skills/`: entrances use ease-out, stay well under Emil's 300ms
ceiling (320ms total incl. fill), and the motion is *purposeful* — it
mimics a shell printing output line-by-line, not decorative fade-in.

**When adding a new line/section:** give it `class="in"` and
`style="--i:<next integer>"`, continuing the sequence from whatever the
current highest `--i` is. Don't reuse `nth-child` delays — the index
approach lets lines be added/reordered without recalculating every sibling.

Always verify a new animation or layout change by rendering with headless
Chrome and reading the screenshot back — don't just eyeball the diff:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu --virtual-time-budget=2000 \
  --screenshot=/path/to/out.png --window-size=1800,1050 --hide-scrollbars \
  "file:///Users/mks/Desktop/C0D3/about-me/index.html"
```
Then `Read` the PNG. Check both the animated-in state (no budget flag) and
the settled end state (`--virtual-time-budget` past ~700ms) plus a mobile
width (~390px) to make sure nothing wraps or clips.

## Content voice — hard-won rules

The user rejected several drafts before landing on the current copy. Do not
drift back toward these rejected patterns:

- **No forced wordplay / cutesy metaphors.** Lines like "the repos tell on
  me", "nightly build", "exit code" were explicitly called out as bullshit.
  Prefer a plain, dry, self-aware statement over a clever pun.
- **No invented identity flourishes.** Don't add "typical", corporate-sounding
  filler, or reach for generic hacker-tinkerer tropes not grounded in the
  user's actual repos/photos.
- **Ground copy in real evidence.** When stuck for a line, check the user's
  actual GitHub repos (`gh repo list MKS-01`) or supplied photos before
  guessing — the "robots, LoRa radios, reverse-engineered wearables, offline
  LLMs" angle came directly from auditing repo names/descriptions.
- **The user writes the real line; you tighten grammar.** Several times the
  final copy was the user's own phrasing verbatim (e.g. the current whoami
  bio) — don't override their wording with something "more creative" once
  they've stated it. Fix wrapping/length/punctuation only.
- Current whoami (do not casually rewrite):
  ```
  senior software engineering lead by day.
  tinkerer at heart — while awake: explore, build, break, repeat.
  ```
  The "explore, build, break, repeat" deliberately echoes the `<title>` tag's
  "coffee → code → tinker → repeat".
- Footer sign-off is a `brew install` pun (Homebrew *and* brewing coffee) —
  `# required dependency` is the punchline; keep the joke self-contained,
  don't stack more jokes onto it.
- Project descriptions are single arrow-lines (`X → Y, detail`), not
  paragraphs. Keep new project entries in that exact terse shape.

## When asked to change design/content

1. Check current repo state (`gh repo list MKS-01`) or ask for the concrete
   input (book title, project name) rather than inventing detail.
2. Propose 2–4 concrete options via AskUserQuestion when the direction is a
   creative/subjective choice (copy, layout style) — this project's owner
   consistently prefers picking from real options over open-ended prose.
3. Make the edit, render + screenshot to verify, then report — don't ask
   "does this look good?" without having looked yourself first.
4. Don't commit unless explicitly asked to.
