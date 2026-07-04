# {{COURSE_TITLE}}

<!--
  One to two sentences. State the topic, the version/API style targeted,
  and the "why trust this" angle (real infra, real tests) if one exists.
  Read as the opening line of a course landing page, not a package
  description — a stranger should know in one sentence whether this is
  for them.
-->
{{ONE_TO_TWO_SENTENCE_TAGLINE}}

<!--
  BADGES. Both of these only resolve once the repo is deployed, so leave
  {{PAGES_URL}} / {{CODESPACES_URL}} / {{PAGES_HOST_PATH}} as literal tokens
  until Deploy step 7 (same tokens the book itself uses in its sidebar —
  see index-template.html/chapter-template.html). Two rules that matter here,
  learned from a real course that got this wrong first:

  1. Never pair a plain markdown link with a graphical badge on the same
     line (e.g. "**[Read the book](url)** · [![badge]]"). The plain link
     and the boxy badge render at different heights/weights and the line
     looks broken, not intentional. Make the book itself a badge too.
  2. Build the "Read the Book" badge as a shields.io badge that matches the
     book's own brand color, so it reads as a matched pair with the
     Codespaces badge rather than a random shield stuck next to GitHub's
     official one:
     https://img.shields.io/badge/📖_Read_the_Book-{{PAGES_HOST_PATH}}-{{BRAND_HEX}}
     ({{BRAND_HEX}} is the course's own --brand CSS variable, no leading
     "#"; {{PAGES_HOST_PATH}} is the Pages host+path with no scheme, e.g.
     "adamontherun.github.io" or "adamontherun.github.io/sqlalchemy-mastery"
     if the Pages URL has a path segment — spaces become underscores, and
     a literal "-" inside a label must be escaped as "--" or shields.io
     reads it as a field separator).

  The Codespaces badge is GitHub's own asset (github.com/codespaces/badge.svg)
  — never reimplement it, it's already the right height to match a default
  shields.io badge.
-->
[![Read the Book](https://img.shields.io/badge/📖_Read_the_Book-{{PAGES_HOST_PATH}}-{{BRAND_HEX}})]({{PAGES_URL}})
[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)]({{CODESPACES_URL}})

<!--
  This paragraph exists because a reader landing on the repo (not the book)
  has no way to know these are two different things with two different
  audiences. Say it plainly. Adapt "terminal with Postgres already running"
  to whatever this course's environment actually provides — a REPL, a
  running dev server, nothing at all if there's no live infra (in which
  case say "a terminal with the toolchain already installed" instead).
-->
This repo is two things: **the book** ({{CHAPTER_COUNT}} chapters of prose,
nothing to install) and **the code** (runnable examples and failing-test
challenges, which need an environment). Every chapter in the book links
straight back to Codespaces, so you're never more than one click from
{{WHAT_S_ALREADY_RUNNING}}.

## What's covered

<!--
  One bullet per PART, not per chapter — a visitor should see the shape of
  the course in five seconds without opening the book. Pull the part
  titles/summaries straight from book/index.html's toc-parts section so
  this never drifts out of sync with the actual table of contents.
-->
- **Part {{PART_ROMAN}} · {{PART_TITLE}}** — {{ONE_LINE_PART_SUMMARY}}

## The book

Open **[book/index.html](book/index.html)** in your browser, or read the
published copy above. It's a self-contained static HTML book: no build
step, works offline, light and dark mode. It doesn't contain any of the
runnable code itself — that lives in {{CODE_DIRECTORIES}} below, which is
why you need one of the environments in "Setup" to actually run anything
the book shows you.

## Setup

<!--
  If stage 1 decided this topic needs live infrastructure, keep both paths
  below as-is. If the topic is self-contained (no Docker/Compose file
  exists), delete the Codespaces-vs-local infra framing and reduce this to
  whatever the ecosystem's own one-line install command is
  (uv sync / npm install / cargo build / etc.) — don't describe an
  environment split that doesn't exist.
-->
Two ways to get a working environment:

**GitHub Codespaces** — click the badge above. It opens a full dev
environment with {{WHAT_S_ALREADY_RUNNING}}. Once it's ready, run
{{POST_CREATE_COMMAND}} if it wasn't run for you automatically, then jump
to "Running examples" below.

**Locally** — you need {{PREREQUISITE_TOOLS_WITH_LINKS}}:

```sh
{{SETUP_COMMANDS}}
```

{{ANY_LOCAL_INFRA_NOTES — e.g. port numbers, credentials, why a non-default
port was chosen. Delete this paragraph if there's nothing non-obvious to say.}}

## Running examples

Every chapter has runnable scripts under `examples/`:

```sh
{{EXAMPLE_RUN_COMMAND}}
```

## Doing challenges

Every chapter has a challenge under `challenges/`: a skeleton file with
failing tests. Edit the skeleton until the tests pass:

```sh
{{CHALLENGE_RUN_COMMAND}}
```

Reference solutions live in `solutions/`. No peeking until the tests pass.

<!--
  Only include this section if the topic has live infra that accumulates
  state examples/challenges don't already clean up after themselves.
  Delete it entirely for self-contained topics.
-->
## Resetting the database

Examples clean up after themselves, but if you ever want a pristine database:

```sh
{{RESET_COMMAND}}
```
