# Markdown link guard skips verbatim research data

`TestEnforcement_MarkdownLinks` failed on `main`: it reported broken `pattern/PATTERN-*.md` links in files under `docs/research/*-2026-10-27/data/`.
Those directories hold bench prompts and reviewer output captured verbatim, and the captured texts link to `pattern/` relative to the repository root, so the links cannot resolve from where the file sits.
The bench landings changed only `docs/`, so no gate ran the guard on their way; every later verify run met the failure.

The guard in `internal/lymcwd/docslink_test.go` now leaves those files out of its scan, and every other markdown file under `docs/` stays guarded.

## Design decisions

- The skip is a path test local to the guard (`docsLinkIsResearchData`), not a `scankit` option; the guard is the only scan that needs it, and `internal/testkit/scankit` is unchanged.
- The skip matches only a `data/` directory directly inside a `docs/research/<report>/` directory.
  A `data/` directory elsewhere under `docs/`, or deeper inside a report, is still scanned.
- Only the source side is skipped.
  Links out of a skipped file go unchecked, but a link from a scanned file into a `data/` file is still resolved, anchors included.
- The allowlist entry for `docs/research/codex-astra-solo-bench-2026-10-25/data/reviews/P4/opus-medium-r1.md` is removed, since the skip covers it and the guard would report it as stale.
- The guard's doc comment and the file header state the exclusion.
  Neither `PATTERN.md` nor any `pattern/` background file states the guard's scope, so no PATTERN text changes.
- A new subtest pins the bound on a `t.TempDir()` fixture.
  A broken link under `docs/research/x/data/` is not reported.
  Broken links in `docs/research/x/report.md`, `docs/other/data/` and `docs/research/x/sub/data/` still are.

## Risk

A future report that keeps its captures under a directory other than `data/` is not skipped and fails the guard until it moves them there.

## Not run by hand

Nothing needs a manual check.
Run `lym gate test ./internal/lymcwd` to confirm the guard and its subtests.
