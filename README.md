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
skeleton). A separate fact-check pass — a fresh subagent with no memory of
drafting the content, re-verifying every citation and flagging uncited
claims — runs after every part, because the agent that wrote a claim is a
poor judge of its own mistake. It was built by actually building a full
course with it first — see [Origin](#origin) below.

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
  the exact same infrastructure a local clone gets, VS Code extensions for
  the ecosystem installed automatically, and the linter wired up as the
  format-on-save formatter from first open.
- Lint, format, and type-checking from the ecosystem's own tools (ruff +
  mypy for Python, eslint + prettier + tsc for JS/TS, clippy + fmt for
  Rust, and so on), set up from the first commit rather than bolted on
  afterward — a real retrofit of this onto an already-finished course
  surfaced 56 lint violations and 17 type errors in one pass, which is
  exactly the pile-up doing it from day one avoids.
- A CI workflow that runs lint/format/type-check plus the challenge suite
  in both directions (must fail against skeletons, must pass against
  solutions) against a real instance of the course's live infra if it has
  any — and, if there's a devcontainer, a second job that builds the
  *actual* `.devcontainer/devcontainer.json` and runs a real example
  inside it, the only automated check that would catch a Codespaces-
  specific bug rather than relying on a one-time manual check.
- Optional one-shot publishing to GitHub: creates the repo under your own
  authenticated account, pushes, enables GitHub Pages, and wires up the
  Codespaces badge.
- The book itself links back out, once deployed: every page's sidebar has
  a one-click "Launch Codespace" button and a GitHub repo link, and every
  chapter's "Try it" box tells the reader they can skip local setup
  entirely and run that chapter's code in Codespaces instead. A reader
  landing on any single chapter page is never more than one click from a
  running environment.
- Full SEO and social metadata on every page (canonical links, keywords,
  Open Graph and Twitter Card tags using a real screenshot of the book as
  the share image), Schema.org `Course`/`Chapter` JSON-LD, a `sitemap.xml`
  and `robots.txt`, and an `llms.txt` following the llmstxt.org convention
  so LLM crawlers get a clean chapter-by-chapter index of the course.
- A screenshot of the book's own cover page, embedded in the README so a
  repo visitor sees what they're getting before reading a description of it.

## Prerequisites

### 1. Claude Code

Pick one:

```sh
# macOS, Linux, WSL — native installer (recommended)
curl -fsSL https://claude.ai/install.sh | bash
```

```sh
# macOS — Homebrew
brew install --cask claude-code
```

```sh
# any platform — npm (requires Node.js 22+)
npm install -g @anthropic-ai/claude-code
```

Windows PowerShell: `irm https://claude.ai/install.ps1 | iex`
Windows WinGet: `winget install Anthropic.ClaudeCode`

Verify: `claude --version`. First run (`claude`) will prompt you to log in
with a Pro/Max/Team/Enterprise/Console account — the free claude.ai plan
doesn't include Claude Code.

### 2. The toolchain your topic needs

The skill detects this per topic and tells you if something's missing, but
having it ready avoids a mid-build interruption. The two most common:

```sh
# uv — for any Python topic
curl -LsSf https://astral.sh/uv/install.sh | sh        # macOS/Linux
# or: brew install uv                                   # macOS
```

Windows: `powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"`
Verify: `uv --version`

```sh
# Node.js — for any JS/TS topic (v22+ recommended)
brew install node          # macOS
# or download from https://nodejs.org
```

Verify: `node --version`

For anything else (Rust, Go, etc.) install that ecosystem's own standard
toolchain — `rustup`/`cargo`, the Go toolchain, and so on.

### 3. Docker — only if your topic needs live infrastructure

Install [Docker Desktop](https://docs.docker.com/get-docker/) (macOS,
Windows, Linux all covered on that page). Verify: `docker --version` and
`docker compose version`.

### 4. GitHub CLI (only if you want to deploy)

```sh
brew install gh                        # macOS
winget install --id GitHub.cli         # Windows
```

Debian/Ubuntu:

```sh
(type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y)) \
	&& sudo mkdir -p -m 755 /etc/apt/keyrings \
	&& out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
	&& cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
	&& sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
	&& sudo mkdir -p -m 755 /etc/apt/sources.list.d \
	&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
	&& sudo apt update \
	&& sudo apt install gh -y
```

Fedora/RHEL: `sudo dnf install dnf5-plugins && sudo dnf config-manager addrepo --from-repofile=https://cli.github.com/packages/rpm/gh-cli.repo && sudo dnf install gh`

Then authenticate — this is the only account setup step, do it once:

```sh
gh auth login --web
```

This opens your browser, you click "Authorize," and it's done. Confirm it
worked:

```sh
gh auth status
```

You should see `✓ Logged in to github.com account <your-username>`. The
skill never asks for or stores a credential of any kind — it only checks
`gh auth status` and reads the logged-in account from it, live, every
time it deploys. If you skip this step, the skill will tell you to run
`gh auth login` and wait rather than trying to deploy anyway.

## Install the skill

Clone this repo directly into your Claude Code skills directory:

```sh
git clone https://github.com/adamontherun/softwarecourse-skill.git ~/.claude/skills/softwarecourse
```

That's it — Claude Code picks up any `SKILL.md` it finds under
`~/.claude/skills/<name>/` automatically. No build step, no dependencies
to install for the skill itself. Confirm it's picked up by starting
`claude` in any project and running `/softwarecourse` — it should show up
as an available skill.

To update later:

```sh
cd ~/.claude/skills/softwarecourse && git pull
```

## Build a course

Start Claude Code (`claude`) in an empty folder for your new course, then
either use the slash command or plain language:

```
/softwarecourse build me a course on Redis
```

```
I want to learn httpx, make me something like a tutorial book with exercises
```

Claude interviews you first (audience level, rough depth, ecosystem,
whether live infra is needed) before writing anything, since those
decisions are much cheaper to get right up front than to redo after twelve
chapters exist. Answer its questions, then it researches, scaffolds, and
builds the course in staged, git-committed steps. Expect this to take a
while for a full course — it behaves like a real, careful multi-hour
project, not a one-shot generation.

## Publish a course to GitHub

Do the [GitHub CLI setup](#4-github-cli-only-if-you-want-to-deploy)
above once, first, if you haven't. Then, once your course is built:

1. In the same Claude Code session (or a new one, from inside the
   course's folder), say:
   ```
   deploy this course to GitHub
   ```
2. Claude checks `gh auth status`. If you're not logged in, it stops and
   tells you to run `gh auth login --web` — it will not proceed without
   your own authenticated session already in place.
3. Claude asks you to confirm the repo name and whether it should be
   public or private (note: GitHub Pages on a private repo requires
   GitHub Pro/Team/Enterprise — use public if you want the site reachable
   on a free account).
4. Claude runs the equivalent of:
   ```sh
   gh repo create <your-account>/<repo-name> --public --source=. --remote=origin --push
   gh api -X POST repos/<your-account>/<repo-name>/pages -f build_type=workflow -f 'source[branch]=main' -f 'source[path]=/'
   ```
   using your live `gh` account, never a hardcoded one, and waits for the
   deploy workflow to actually finish (not just for these commands to exit).
5. You'll get back two URLs: the repo
   (`https://github.com/<your-account>/<repo-name>`) and the live book
   (`https://<your-account>.github.io/<repo-name>/`) — both already
   verified reachable before Claude tells you it's done. No manual step
   in GitHub's web UI is required; enabling Pages via the API above
   handles what you'd otherwise click through in Settings → Pages.

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
  readme-template.md            repo-root README starting point, incl.
                                  the book/Codespaces badge-pairing rules
  robots-template.txt           robots.txt starting point
  sitemap-template.xml          sitemap.xml starting point (one <url>
                                  per chapter, generated at deploy)
  llms-template.txt             llms.txt starting point (llmstxt.org)
  devcontainer.json              generic devcontainer template, incl. VS
                                  Code extensions/settings per ecosystem
  docker-compose.extend.yml      layers dev tooling onto a course's own
                                  compose.yaml without editing it
  pages-deploy.yml               GitHub Pages deploy workflow
  ci-template.yml                lint/format/type-check/test CI, incl. a
                                  devcontainer-build smoke test
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
