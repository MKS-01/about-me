# Custom skills

Project-scoped skills for building & reviewing this landing page. Anything
placed here is picked up automatically by Claude Code when working in this repo.

Each skill is its own folder with a `SKILL.md` (name + description frontmatter,
then the instructions). See `.agents/skills/` for the installed design skills
(Emil Kowalski's design-engineering set) used to tune the page's motion & polish.

- **portfolio-conventions/** — this project's own design system, content
  voice rules, and animation conventions for `index.html`. Read this before
  any layout or copy change; it captures decisions already made (and
  rejected) so they don't get re-litigated or reverted by accident.
- **writing-a-blog-post/** — the `blog/*.html` format: the `less(1)`
  conceit, the prose markup that's already styled, read-time, and the
  two-file publish step. Read this before drafting or editing a post.
  Assumes `portfolio-conventions` for the underlying design system.
