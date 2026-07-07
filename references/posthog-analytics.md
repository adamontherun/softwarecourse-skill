# Optional reader analytics (PostHog)

Every generated course ships with an inert PostHog module in
`assets/book.js`. It stays **off** — no key, no network request — until three
tokens near the top of that file are filled:

```js
var PH_KEY  = "{{POSTHOG_KEY}}";          // public project token, phc_...
var PH_HOST = "{{POSTHOG_HOST}}";         // https://us.i.posthog.com | https://eu.i.posthog.com
var COURSE  = "{{POSTHOG_COURSE_SLUG}}";  // e.g. "redis-in-depth"
```

The guard `if (PH_KEY && PH_KEY.indexOf("{{") !== 0)` means an unresolved or
blank key disables the whole module. This is deliberate: the skill is public,
so nothing tracks readers unless a course author explicitly opts in.

## Which key is safe

- **Project API key (`phc_…`)** — designed to be embedded in client-side
  pages; it's the same token PostHog's own install snippet uses. **This is the
  only value that belongs in `book.js`.** It can only *send* events, not read
  data, so publishing it (including in a public GitHub repo / GitHub Pages) is
  expected and safe.
- **Personal API key (`phx_…`)** and any project *secret* — **never** put these
  in a course. They can read/modify your PostHog data. If one ever lands in a
  committed file, rotate it in PostHog immediately.

## Creating the account (one-time)

1. Sign up at **https://posthog.com** (free tier is generous).
2. Choose a **cloud region** — **US** (`https://us.i.posthog.com`) or **EU**
   (`https://eu.i.posthog.com`). Data is *not* shared between regions, so pick
   deliberately; the region you pick is the `PH_HOST` value.
3. Create a project (one shared project can host many courses — they're
   separated by the `course` property below).
4. **Project settings → Project API key** → copy the `phc_…` value into
   `PH_KEY`. Set `PH_HOST` to your region host.
5. Set `COURSE` to a short slug unique to this course.
6. Deploy, open a chapter, then check **Activity → Live events** in PostHog —
   events should appear within seconds, each carrying `course`.

## What gets tracked

Basic web analytics come free from PostHog's autocapture + the dated
`defaults` snapshot: `$pageview`, `$pageleave` (→ time-on-page / reading time),
autocaptured clicks, and rageclicks. `person_profiles: "identified_only"`
keeps anonymous readers cheap and privacy-light — no person profiles are
created and the course never calls `identify`.

Custom events added on top (all carry `course`, and on chapter pages also
`chapter_number`, `chapter_title`, `part`):

| Event | Fires when | Key properties |
|-------|-----------|----------------|
| `$pageview` | Each page load | `page_type` (`home`/`chapter`), chapter context |
| `codespace_launch_clicked` | A "Launch Codespace" link is clicked | `location` (`sidebar`/`tryit`/`hero`), `href` |
| `repo_link_clicked` | The "GitHub repo" link is clicked | `href` |
| `section_viewed` | An `<h2>`/try-it heading scrolls into view (once each) | `section`, `section_index` |
| `scroll_depth` | Reader passes 25/50/75/100 % of the page | `depth` |
| `code_copied` | A code block's copy button is used | `label` (filename) |
| `theme_changed` | Light/dark toggle | `theme` |

## Suggested insights to build

- **Codespace funnel** — `$pageview` → `codespace_launch_clicked`, broken down
  by `course` and `location`. Answers "who actually launches the environment,
  and from where."
- **Chapter drop-off** — `section_viewed` count by `section_index` within a
  `chapter_number`; where the line falls off is where readers stop.
- **Most-read chapters** — `$pageview` (or `$pageleave` duration) by
  `chapter_title`.
- **Most-used examples** — `code_copied` by `label`.
- **Cross-course comparison** — any of the above broken down by `course`.

## Privacy notes

No PII is collected, no session recording or heatmaps are enabled, and
`identified_only` avoids creating person profiles for anonymous readers. That
keeps this comfortably in "anonymous product analytics" territory. If a course
targets an audience where an explicit consent banner is warranted, PostHog
supports opt-in capture (`opt_out_capturing_by_default` / `opt_in_capturing`) —
wire a small banner and gate the `posthog.init` call behind consent.
