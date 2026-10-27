# Decision record — docs-link-skip-bench-data

## Goal

`TestEnforcement_MarkdownLinks` passes on `main` again by leaving the verbatim data of research reports out of its scan, while every other markdown file under `docs/` stays guarded.

## Scope

- `internal/lymcwd/docslink_test.go`: the scan skips every `.md` file under a `data/` directory that sits directly in a `docs/research/<report>/` directory.
- The allowlist entry for `docs/research/codex-astra-solo-bench-2026-10-25/data/reviews/P4/opus-medium-r1.md` is removed.
- A subtest pins the skip.
- The guard's doc comment states that `docs/research/*/data/` is outside its scope.
- Out of scope: rewriting any link inside the captured data, and any change to `internal/testkit/scankit`.

## Decisions

### The skip is a path test inside the guard, not a scankit option

Decision: `docsLinkScan` skips a file whose base-relative path has the shape `docs/research/<report>/data/...` before extracting its links.
Rationale: the guard is the only scan that needs the skip, so it stays local to the guard, as the code-quality rule against a premature shared option asks.
Bound: the skip lets through only files whose path starts with exactly `docs`, `research`, one report directory and `data`; a `data/` directory anywhere else under `docs/`, or deeper inside a report, is still scanned.

### Only the source side is skipped

Decision: a skipped file's outgoing links go unchecked, but a link from a scanned file into a `data/` file is still resolved, anchors included.
Rationale: the captured data is never rewritten, so its own links may break; the report text that points into it is documentation and must still resolve.

### The guard's scope is stated in its doc comment only

Decision: the doc comment of `TestEnforcement_MarkdownLinks` and the file header carry the exclusion.
Rationale: neither `PATTERN.md` nor any `pattern/` background file states the guard's scope today, so there is no PATTERN text to amend.

## Constraints

- `PATTERN-test-speed`: the subtest stays in the existing `tier0` file and runs against a `t.TempDir()` fixture.
- `PATTERN-test-isolation`: the fixture tree is built under `t.TempDir()`, never in the live repository.
- The fixture paths never contain `testdata`, since scankit skips such directories and the subtest would pass vacuously.

## Auto-mode assumptions

- "A `data/` directory of a `docs/research/<report>/` directory" means the direct child `data/` of the report directory, as every current report lays it out.
- The board entry's PATTERN text clause is satisfied by the doc comment alone, since no PATTERN text states the guard's scope.

## Open risks

- A future report that stores its captures under another directory name is not skipped and fails the guard until it moves them under `data/`.

## Acceptance criteria

- A broken link in `docs/research/x/data/<file>.md` is not reported by the scan.
- A broken link in `docs/research/x/report.md` is still reported.
- The repository scan reports no breaks and no stale allowlist entry.
- The removed allowlist entry is gone from `docsLinkAllowlist`.
