# softwarecourse

A [Claude Code](https://claude.com/claude-code) skill that builds a complete,
research-grounded course/book teaching a specific software topic — a
library, framework, language feature, protocol, or tool you name. It
produces a hand-crafted, mobile-friendly HTML5 book with runnable and
verified code examples, test-driven challenges with reference solutions,
and — if you ask it to — publishes the whole thing to GitHub with Pages
and Codespaces support.

This isn't a template generator. Every factual claim gets checked against
real documentation before it's written down, every code example actually
gets run (never fabricated output), and every challenge's tests get
verified both ways (pass against the solution, fail against the untouched
skeleton). It was built by actually building a full course with it first —
see [Origin](#origin) below.

## What you get

- A full HTML5 book: cover page, per-chapter pages, sidebar table of
  contents, light/dark mode, mobile-responsive layout, callout boxes for
  gotchas/production notes/deep-dives, syntax-highlighted code blocks with
  copy buttons — all hand-crafted CSS/HTML, no framework, no build step.
- Inline citations linking load-bearing claims (gotchas, best practices,
  quoted docs) to their real source URLs, not just an internal bibliography.
- Runnable example scripts and a test-driven challenge (skeleton + tests +
  reference solution) per chapter, in whatever toolchain the topic's own
  ecosystem actually uses — uv/pytest for Python, npm/vitest for JS/TS,
  cargo for Rust, and so on.
- Live infrastructure via Docker Compose when the topic genuinely needs a
  real service (a database, a broker) — and *not* when it doesn't.
- A `.devcontainer/` setup so the course opens in GitHub Codespaces with
  the exact same infrastructure a local clone gets.
- Optional one-shot publishing to GitHub: creates the repo under your own
  authenticated account, pushes, enables GitHub Pages, and wires up the
  Codespaces badge.

## Prerequisites

- [Claude Code](https://claude.com/claude-code)
- [uv](https://docs.astral.sh/uv/) or the relevant toolchain for whatever
  topic you ask for (the skill figures out which one per topic)
- [Docker](https://docs.docker.com/get-docker/), only if your topic needs
  live infrastructure
- [GitHub CLI](https://cli.github.com/) (`gh`), only if you want the
  publish/deploy step — and only if you're already logged in
  (`gh auth login`). The skill never asks for or stores a credential of
  any kind; it only checks whether `gh` is already authenticated and uses
  whatever account that resolves to, live, every time. It cannot deploy on
  your behalf without your own `gh` login already in place.

## Install

Clone this repo directly into your Claude Code skills directory:

```sh
git clone https://github.com/adamontherun/softwarecourse-skill.git ~/.claude/skills/softwarecourse
```

That's it — Claude Code picks up any `SKILL.md` it finds under
`~/.claude/skills/<name>/` automatically. No build step, no dependencies
to install for the skill itself.

To update later:

```sh
cd ~/.claude/skills/softwarecourse && git pull
```

## Use

In a Claude Code session, just ask for what you want:

```
/softwarecourse build me a course on Redis
```

or without the slash, plain language works too — anything like "make me a
tutorial book on React Server Components" or "I want to learn httpx,
create something like a course" will trigger it.

Claude will interview you first (audience level, rough depth, ecosystem,
whether live infra is needed) before writing anything, since those
decisions are much cheaper to get right up front than to redo after twelve
chapters exist. From there it researches, scaffolds, and builds the course
in staged, git-committed steps — this can take a while for a full course
(expect it to behave like a real, careful multi-hour project, not a
one-shot generation).

To publish an already-built course:

```
deploy this course to GitHub
```

This only runs when you ask for it — creating a public repo and pushing
is a visible action, not something that happens as a side effect of
building content.

## Repo structure

```
SKILL.md                       the skill itself: the full staged workflow
assets/
  style.css, book.js           the reusable design system, copied verbatim
                                into every course (mobile-layout bugs
                                already fixed — see the comments in
                                style.css near the 900px breakpoint)
  vendor/highlight.min.js       vendored syntax highlighter, no CDN needed
  chapter-template.html         structural pattern for a chapter page
  index-template.html           structural pattern for the cover/TOC page
  devcontainer.json              generic devcontainer template
  docker-compose.extend.yml      layers dev tooling onto a course's own
                                  compose.yaml without editing it
  pages-deploy.yml               GitHub Pages deploy workflow
references/
  humanizer-checklist.md         33 patterns that make prose read as
                                  AI-generated, applied as an explicit
                                  editing pass on every course's content
evals/
  evals.json                     the test prompts used to validate this
                                  skill's early-stage behavior
```

## Origin

This skill was extracted from a real session building a complete
SQLAlchemy course end to end — see it live at
[adamontherun.github.io/sqlalchemy-mastery](https://adamontherun.github.io/sqlalchemy-mastery/)
([source](https://github.com/adamontherun/sqlalchemy-mastery)). Every
pattern this skill encodes — the design system, the mobile-responsive
fixes, the humanizing pass, the citation style, the devcontainer/Pages
setup — was built and verified against that real course first, then
generalized. It isn't a hypothetical best-practices document; it's what
already worked once, written down so it doesn't have to be re-derived
every time.

## License

MIT — see [LICENSE](LICENSE).
