# Markdown link guard skips the verbatim data of research reports

`TestEnforcement_MarkdownLinks` failed on `main` on files under `docs/research/*/data/`.
Those directories keep bench prompts and reviewer output verbatim, and the captured texts link to `pattern/PATTERN-*.md` relative to the repository root, so the links never resolve from where the file sits.
The bench landings changed only `docs/`, so no gate on their way ran the guard.

## What changes

- `docsLinkScan` in `internal/lymcwd/docslink_test.go` no longer extracts links from a file under the `data/` directory of a `docs/research/<report>/` directory.
  The predicate `docsLinkIsResearchData` matches only that exact shape: `docs`, `research`, one report directory, then `data`.
  A `data/` directory anywhere else under `docs/` is still scanned.
- The skip is source-side only: a link from a report into its own `data/` files is still resolved, anchors included.
- The allowlist entry for `docs/research/codex-astra-solo-bench-2026-10-25/data/reviews/P4/opus-medium-r1.md` is removed, since the skip covers it and the guard would otherwise report it stale.
- A new subtest builds a fixture with the same broken link in `docs/research/x/data/prompts/step.md` and in `docs/research/x/report.md`, and asserts that only the report's link is reported.

## Design decisions

- The skip lives in the guard, not as an option on `internal/testkit/scankit`, because no other scan needs it.
- The guard's scope is stated in the file header and in the doc comments of `docsLinkScan` and `TestEnforcement_MarkdownLinks`.
  Neither `PATTERN.md` nor any `pattern/` background file states the guard's scope, so no PATTERN text changes.
