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

- The page is a **scroll-snap deck** (`scroll-snap-type: y mandatory` on
  `html`): screen 1 is the landing (`#home`), then one full-viewport
  `.screen.detail-screen.reveal` per project. Each `.screen` is
  `min-height: 100dvh` flex-centered with `padding: 3rem 1.5rem`; inner
  wrapper `.inner` is `max-width: 720px` (940px on home + detail screens,
  whose text column `.content` caps at 540px so it clears the fixed mac).
  Never go back to a top-anchored layout with huge bottom whitespace —
  explicitly rejected ("looks like hanging on the top").
- Every section is a `<p class="prompt">` (green `$ command`) followed by
  `<div class="output">` / a result paragraph. New content should follow this
  exact pattern, not a departure into card/grid layouts.
- The header alien is an **inline SVG** (Noto Emoji U+1F47D paths, body
  recolored `var(--accent)`, eyes/mouth `var(--bg)`), duplicated as a
  hardcoded-hex data-URI favicon so every OS shows the identical mark. The
  `👽` emoji was standardized away on purpose — don't reintroduce it.
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

**Headless Chrome gotchas (learned the hard way):**
- Screenshots of fragment URLs (`index.html#pizow`) render blank — even for
  trivial pages. To verify below-the-fold screens, `sed` a test copy that
  replaces `min-height: 100dvh` with a fixed px height and pre-adds `on` to
  the `.reveal` sections, then screenshot the whole stack in one tall window.
- Window width clamps to 500px minimum (old and new headless). A
  `--window-size=390,…` screenshot is a 390px crop of a 500px layout — fake
  "overflow". To test true mobile width, constrain `body { width: 390px }`
  in the test copy, or probe `getBoundingClientRect().right > 391` from an
  injected script and read it back via `--dump-dom`.
- The `.mac` is `position: fixed`, so in a tall stacked-test window it sits
  at the bottom of the *whole* render, not per screen — crop the bottom band
  to inspect it. To force a specific app/lid state in a test copy, `sed` the
  markup to `class="mac open" data-app="pizow"` **and** strip the JS lines
  that overwrite it (`setLid`-style init call and
  `mac.dataset.app = e.target.id`), or the observer resets your forced state
  on load.

## Scroll experience & the mac

Architecture that emerged over many iterations (see history below before
proposing alternatives):

- **Reveal**: each detail screen restarts its `--i` print sequence at 0; its
  lines are `animation-play-state: paused` until an IntersectionObserver
  adds `.on` on first view. Keep the `<noscript>` fallback that unpauses.
- **Detail content grammar** matches the landing exactly: `$ cat
  ~/weekend-hacks/<name>/README.md` prompt → arrow-line title → three `›`
  points → `stack →` line → `$ git clone` / `$ open` cmdlines. Copy must be
  grounded in the real repo description/topics (`gh repo view`).
- `.prompt`/`.cmdline` have `overflow-wrap: anywhere` so long paths wrap
  like a real terminal on mobile — don't remove it.
- **The mac is the page's centerpiece gimmick.** A CSS-drawn open MacBook
  (`.mac`: lid + camera dot + base with notch), `position: fixed` on the
  right (`left: calc(50% + 90px)`, 400px wide, ≥1021px viewports). Its
  screen hosts one `.app` per section (`app-home` terminal lines,
  `app-readback` EQ bars + progress, `app-pizow` htop gauges, `app-mlx`
  gpu bar + typewriter). The same IntersectionObserver that drives the
  rail sets `data-app = screen id`, crossfading apps as you scroll. All
  demo animation is CSS-only; the mac is `aria-hidden`.
- **Lid behavior: ALWAYS open, from first paint.** The markup hardcodes
  `class="mac open"`; there is no lid animation. Three earlier behaviors
  were built and rejected in sequence: scroll-position-linked lid angle
  ("accuracy not coming — minimum scrolling space"; snap gives no mid-scroll
  dwell), open-on-scroll/close-at-top ("lets the lid open only"), and a
  delayed open-once-on-load swing (owner, angrily: "I asked you to keep the
  lid open no effect on first load"). Do not reintroduce any lid motion.
  The closed-lid CSS (`rotateX(-84deg)` without `.open`) still exists but
  must never be user-visible.
- **Section nav is a tmux status bar** (`nav.rail`, restyled July 2026):
  thin bar pinned to the bottom edge, `[mks]` session name in green, each
  screen a numbered window (`0:home* 1:readback …`), active one accent +
  starred (star width is pre-reserved so the bar never shifts). Driven by
  the same observer. `© year mks` (`.site-foot`) lives in its right corner
  like tmux right-status; session name + © hide ≤640px, tighter gap/font
  ≤440px (fits 360px), safe-area padded. It replaced a right-edge rail of
  square dots — chosen by the owner over vim-statusline, prompt-history
  rail, and ASCII scrollbar options.
- **Mobile (≤1020px)**: the mac docks `position: fixed; bottom` centered,
  scaled 0.75 (0.68 ≤440px), and is hidden (`opacity: 0`) while
  `data-app="home"` so the landing stays a clean terminal; detail screens
  get `padding-bottom: 220px` so text clears it. The desktop `fadeIn`
  entrance is replaced by an opacity transition in this mode.
- Reduced motion: lid always open, apps swap instantly, all loops off.

**Design history for the right side (don't re-propose losers):** giant faint
emoji glyphs — built, then reverted ("not looking good"); static ASCII flow
diagrams — rejected unseen ("too boring"); bordered tmux-style demo panes
beside each project — loved ("damm this looks coool") but later folded into
the mac's screen at the owner's idea. Lesson: filler must be *alive and
specific to the project*, not decorative. Cold-brew glass beside the mac
(CSS-drawn tumbler, floating ice, level dropped + ice melted with scroll) —
iterated through many rounds (icons → big cubes → bottom pile → floating
per physics; brown → darker brown → accent blue) and ultimately **removed**:
owner felt coffee "doesn't fit the theme" and cut it even in on-palette
blue. Don't re-propose desk props next to the mac; the footer's
`brew install cold-brew` pun carries the coffee joke alone.

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
