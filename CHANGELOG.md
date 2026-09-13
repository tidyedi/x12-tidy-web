# Changelog

## Unreleased

Changes on `main` since the v0.2.0 tag, not yet cut as a new version.

## v0.2.0 — 2026-09-13

### Deployed

- **The canonical instance is live.** Landing page at
  <https://repair.tidyedi.com> (GitHub Pages, `main` `/docs`); the app on
  Render (Docker, `render.yaml`, Starter plan — always-on). Worked examples at
  <https://repair.tidyedi.com/demo/>. Deployment and ops detail moved to a
  private repo.
- Static assets are linked root-relative and `serve` trusts the platform's
  `X-Forwarded-Proto`, so the app renders correctly behind Render's
  TLS-terminating proxy (previously served `http://` asset URLs that the
  browser blocked as mixed content). This also makes the per-IP rate limit
  key on the real client IP.

### Changed

- **The verdict is decided in one place** (`RepairRun.verdict`) and rendered by
  the app, the report, and the demo alike. Any residual **fatal** finding now
  reads as "Cannot be repaired", and the payload is labelled a *partially
  repaired interchange — not conformant* with a caveat, never a "corrected
  interchange".
- The "Repair passes" section and every report format now explain the pass
  model (pass 1 repairs your input; later passes run on the prior output).
- Reports say "Why it stopped: <plain sentence>" instead of the internal
  `stop_reason` key.
- `CLAUDE.md`'s "one rule" reworded as maintainer-facing guidance about the
  x12-tidy import boundary.
- **The repo is renamed `web-facing` → `x12-tidy-web`** to match the package,
  CLI, and Render service, which already used that name. GitHub redirects the
  old URLs; <https://repair.tidyedi.com> and its `/demo/` path are unaffected.
- **fatal / error / warning each have one definition now** — a shared
  `_severity_legend.html` partial, rendered identically under the repair passes
  and on `/codes`, replacing three looser phrasings (one of which wrongly called
  an *error* "advisory"). The wording was checked against every code in the
  installed registry (30 fatal / 10 error / 5 warning) so each definition holds
  for its whole bucket — "fatal" now covers envelope self-check failures
  (counts, control numbers, version), not just "a conforming parser rejects it".
  It only *describes* x12-tidy's classification — the registry still assigns
  severity — and it lives in the template, not in code. Classification questions
  raised while doing this went to x12-tidy as a maintainer reminder
  (`tidyedi/x12-tidy#87`), not as a fix.
- **"Feedback" (nav, footer, landing page) is now an email link**
  (`repair-feedback@tidyedi.com`) when a feedback address is configured — the
  no-account path — falling back to a GitHub discussion otherwise. The
  post-repair prompt is a clearer callout and pre-fills the run's verdict,
  version, and stop reason into the message.
- The EDI textarea's placeholder tells a first-time visitor what to do
  instead of only showing raw EDI shape, and a plain page reload now clears
  the saved draft (an in-tab trip to `/codes` and back still restores it).
- The verdict banner (clean / residual / fail) now gets the same visual
  treatment as the privacy callout — circular icon badge, thicker accent bar,
  stronger tint, shadow — and each state's color matches its corresponding
  severity pill exactly, so the banner and the pills read as one consistent
  color language.
- Severity descriptions on a pass moved from an always-visible legend under
  every pass to a hover tooltip on each chip, with an explicit "hover a chip"
  hint so the affordance isn't silent (live app + static demo).
- The clean verdict now distinguishes "already conformant; nothing needed
  fixing" from "repaired and now conformant, with nothing left to fix" —
  previously it read the same either way, which contradicted the sub-line
  when earlier passes had, in fact, changed bytes.
- The ISA fixed-width padding note is now a soft-tinted callout that leads
  with "This is expected, not an error:" instead of an easy-to-miss gray hint
  line — deliberately without the bordered/accent-bar/icon-badge shape the
  app's actual alerts use, so it reads as reassurance, not a failure.
- The verdict now repeats at the end of the pass list, so a visitor who
  scrolled through several passes doesn't have to scroll back to the top to
  see the final status. The downloadable report moved from a small
  select+button tucked into the "Repair passes" heading to its own callout at
  the end of the results, and gained a **View** button (opens in a new tab)
  alongside **Download**.
- The downloaded HTML report now carries the app's favicon and links
  "x12-tidy-web" to <https://repair.tidyedi.com> and "x12-tidy" to its exact
  source commit, so a saved or forwarded report still shows where it came
  from.

## v0.1.0 — 2026-09-06

First tagged release. A stateless web front end for
[x12-tidy](https://github.com/tidyedi/x12-tidy): paste a malformed ANSI X12
interchange, get an iteratively-repaired copy and a per-pass report.

### The app

- **Iterative repair** (`engine.repair`) — runs `x12_tidy.tidy()` to a fixed
  point, one `Iteration` per pass, stopping on clean / stable / unrecoverable /
  max-iterations.
- **The form** — paste or upload an interchange, or pick from five broken
  samples; get a verdict, the corrected interchange, the envelope facts, and
  every pass's findings. Filter a pass's findings by severity. Read the
  corrected bytes laid out segment-by-segment. Download the run as JSON,
  Markdown, HTML, text, or CSV.
- **Verdicts** are colour-coded: green for clean, amber for repaired-with-
  residual-findings, red for unrecoverable or not-repairable.
- **`/codes`** — a reference for every diagnostic code the installed x12-tidy
  can emit, grouped by area, filterable by severity, each row linking to a
  pre-filled discussion on x12-tidy.
- **JSON API** — `POST /api/validate`, `POST /api/report`, `GET /api/formats`,
  `GET /api/codes`, `GET /healthz`.
- **CLI** — `x12-tidy-web serve | repair FILE | demo [OUT_DIR]`.

### Provenance and the one rule

- All X12 knowledge lives in x12-tidy and is imported, never copied.
  `tests/test_dependency_provenance.py` fails if that is ever violated.
- Every result is stamped with the exact x12-tidy git commit — footer,
  `/healthz`, `/api/codes`, `--version`, and downloaded reports.

### Operations

- Per-IP rate limiting on the two repair endpoints (`slowapi`;
  `X12_TIDY_WEB_RATE_LIMIT`).
- `serve` honours `$PORT`; the Docker image runs read-only, non-root.
- Opt-in "report a wrong result" `mailto:` (`X12_TIDY_WEB_FEEDBACK_EMAIL`).
- Static demo bundle in `docs/demo/`, published on GitHub Pages.
- Deployment guide in `docs/DEPLOYMENT.md` (later moved to a private ops repo;
  see the Unreleased section).

### Not yet done

An update mailing list and a few marketing/links items.
