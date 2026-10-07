---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/planindex/planindex.go:12
    summary: New and edited Go comments break sentences at a fixed column, which PATTERN-comment-line-breaks forbids and lyx loom lint-comments flags at 19 lines
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/commentlint/lint_integration_test.go:46
    summary: Four new scenario and guard tests omit t.Parallel and carry no comment naming global state, against PATTERN-test-speed
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/commentlint/lint.go:121
    summary: A directive line anywhere in a comment block exempts the whole block, so the lint misses wrapped prose above every //lyx:guard and //testtiming:keep marker
  - id: F4
    severity: LOW
    class: design
    location: internal/commentlint/lint.go:28
    summary: The conjunction set lacks yet and so, so a comma before those coordinating conjunctions is flagged although PATTERN-comment-line-breaks allows it
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/loomshed/verifygate.go:62
    summary: Every impactset.Derive error becomes a writer finding with no way forward, though git and directory read failures are not defects the writer can fix
  - id: F6
    severity: BLOCKING
    class: consistency
    location: contracts/specs/refusal-spec.md:223
    summary: The Webster-Burler verify gate row names only failing tests, but the gate now also refuses on comment lint findings and misplaced guard markers
  - id: F7
    severity: LOW
    class: design
    location: cmd/lyx/testdata/test-budget.yaml:4
    summary: The cmd/lyx budget row is 70 while the package holds 69 top-level tests, so the budget is not pinned at the after-state and one test can be added unseen
  - id: F8
    severity: BLOCKING
    class: scope
    location: internal/websterengine/geometry.go:80
    summary: The nil Index error path that Geometry.index adds has no test reaching it
  - id: F9
    severity: BLOCKING
    class: consistency
    location: cmd/lyx/treesitterlink_test.go:151
    summary: TestParseGoListImports tests an unexported helper that TestTreeSitterLink_OnlyAllowedTestBinariesLinkIt already covers over real go list output
---

Range: no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, that is `d536a1a4e..HEAD`, cards 1 to 15 (`product.parent` is `main` in `_lyx/shed/test-tiers/status.json`).

Verified on the substrate: `go build ./...`, `go vet ./...`, `go test ./...` and `go test -tags integration ./...` all pass on the tree as committed.
`lyx loom lint-comments --base d536a1a4e`, built from this branch, reports 19 lines (F1).

Settled per focus entry:

- Cards 6, 7, 10 (lint): an untouched wrapped block, an untouched wrapped line next to an added break, a pure rename, and a fixed break with the block's other breaks left alone are all correctly unflagged, and the `.`/`!`/`?`/`:` plus closing `)`/quote/backtick, semicolon and comma-plus-`and`/`but`/`or`/`nor` breaks pass.
  Two defects: F3 (a missed wrap) and F4 (a legitimate break flagged).
  The base-text pair key is file-global, so a new break whose two boundary words also meet at some base line end is missed; the plan prescribes exactly that key, so it is not filed.
- Cards 8, 10 (fallbacks): an unreadable or missing record, an empty base, a non-ancestor base, a changed `go.mod`/`go.sum`, a changed non-Go file outside every package, a deleted file whose directory is no longer a package, a failing `go list` and an over-long command all end in the plan's verify command, never an empty or narrower one.
  A `//lyx:guard` test always joins the command by name.
  The one weak path is F5 (an error from `Derive` is a finding, not a fallback).
- Card 1: records are matched by exact command, the base entry survives pruning through `BaseCommand`, and an old-format or malformed record reads as none; a failed run writes nothing, so `LatestPass` only ever returns a recorded pass, and a base that is stale only widens the diff.
  No defect.
- Cards 2 to 5: `go list -deps` of `websterengine` and `loomshed` shows no `planglyph`, `quarry` or `go-tree-sitter`; the eight test binaries linking tree-sitter match the allowed list in the guard, and the guard passes in both directions.
  F9 is the one finding on the guard's tests.
- Card 12: an absent key loads the template's empty value, a failing `publish_verify` halts Stuck naming the key, the exit code and the log, an empty value runs and logs nothing, and Finalize never runs it.
  No defect.
- Card 14: the count is derived by a static parse and every package row but `cmd/lyx` equals the repository's count (F7).
- Per-card mechanical check: no card carries a `Rename` or `Delete` group, so no type-specific script applies; every `Create` target (`LatestPass`, `Index`, `NewIndex`, `Lint`, `Derive`) exists.
  Card 9: every top-level test that walks the repository outside its own package directory carries `//lyx:guard`; the unmarked scans read only their own package directory.
- Test economy: F8 and F9.

### [BLOCKING] Fixed-column comment breaks in new and edited comments

**Location:** `internal/planindex/planindex.go:12-24,38-39,50-52`, `internal/planglyph/repo.go:2,20`, `internal/websterengine/beginbatch.go:74,115`, `internal/websterengine/runlevel.go:207`.
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line with no column limit; these comments break mid-sentence at about column 100.
The `ErrQuarryUnavailable`, `Finding` and `Error` comments in `planindex.go` were moved and edited, so every line of them is new text.
**Fix:** Rewrap each sentence onto its own line (a sentence that is long keeps its internal break only at a semicolon or a comma before a conjunction), then rerun `lyx loom lint-comments --base d536a1a4e` until it is clean.

### [BLOCKING] Scenario and guard tests run serially without naming global state

**Location:** `internal/commentlint/lint_integration_test.go:46`, `internal/impactset/derive_integration_test.go:42`, `internal/loomcli/lintcomments_integration_test.go:20`, `cmd/lyx/treesitterlink_integration_test.go:34`.
**Issue:** `PATTERN-test-speed` requires `t.Parallel` unless the test touches process-global state and a comment names it.
`TestLint_OverARepository`, `TestDerive_OverARepository`, `TestLintCommentsCmd_OverARepository` and `TestTreeSitterLink_OnlyAllowedTestBinariesLinkIt` touch only a temp directory or run a read-only `go list`, and call neither `t.Parallel` nor name a global.
**Fix:** Call `t.Parallel()` first in each, with the sentence the neighbouring scenario tests use: the steps share one repository, so the test is parallel as a whole and no step is.

### [MEDIUM] A directive line exempts its whole comment block from the lint

**Location:** `internal/commentlint/lint.go:121` (`isProseBlock`), `internal/commentlint/doc.go:17`.
**Issue:** `isProseBlock` rejects a block when any line is a directive, and card 9 stacks `//lyx:guard` and the existing `//testtiming:keep` into the doc comment block of every guard and kept test.
A wrapped doc comment above such a marker is therefore never flagged: in a scratch repository, a wrapped `TestY` doc is reported and the same doc above `//lyx:guard` is not.
**Fix:** Make a directive line end the block it sits in, and keep the prose lines above it checked; indented code, list and heading lines keep skipping the whole block.
Update `doc.go`, and turn the "directive comment block is never flagged" row into a row showing prose above a directive is still checked.

### [LOW] Coordinating conjunctions `yet` and `so` are not recognised

**Location:** `internal/commentlint/lint.go:28`.
**Issue:** The set holds `and`, `but`, `or` and `nor`; a line ending in a comma before `yet` or `so` starts an independent clause and is an allowed break under `PATTERN-comment-line-breaks`, yet it is flagged.
**Fix:** Add `yet` and `so`, leave `for` out because `, for example` is introductory, and add a row to the existing comma-and-conjunction case in `TestFindNewWrappedBreaks`.

### [MEDIUM] Every `Derive` error is handed to the writer as a finding

**Location:** `internal/loomshed/verifygate.go:62`, `internal/impactset/guards.go:35`.
**Issue:** The gate's doc says a returned error is one "the writer can fix" is not, yet a failed `git diff`, a failed directory read and a misplaced marker all become `Passed: false` with "The round command could not be derived: <err>".
A git or directory failure then burns the gate's attempts, and no message names the way forward (`PATTERN-refusal-way-forward`).
**Fix:** Wrap the marker, parse and tier-constraint errors of the guard scan in an exported `impactset.ErrGuardScan`; the gate turns those into a finding that names the file, the line and the move that fixes it (put the marker directly above the func line, keep it out of `tmux` and `llm` files), and returns every other `Derive` error as an error.

### [BLOCKING] The verify-gate refusal row does not cover the lint and marker refusals

**Location:** `contracts/specs/refusal-spec.md:223`.
**Issue:** The Webster-Burler verify gate now also refuses on a comment lint finding and on a misplaced `//lyx:guard` marker, but its row says "fix the failing tests ... check them with `lyx webster verify`".
`PATTERN-refusal-way-forward` requires a refusal's row and way forward to land with it.
**Fix:** Widen the row's condition to the lint and marker refusals and name `lyx loom lint-comments` as their check.

### [LOW] The `cmd/lyx` budget has a spare test

**Location:** `cmd/lyx/testdata/test-budget.yaml:4`.
**Issue:** The row says 70 and the package holds 69 top-level tests under every tag (`go test -tags 'integration tmux llm' -list` agrees), so the budget is not pinned at the after-state and a 70th test passes unseen.
**Fix:** Set the row to the count left after F9, and confirm every other row still equals its count.

### [BLOCKING] The nil Index error has no test

**Location:** `internal/websterengine/geometry.go:80`.
**Issue:** Card 3 adds that a nil `Index` reached at a call is a returned error naming the missing wiring, never a panic; `Geometry.index` and `errNoIndex` have no test reaching them (`PATTERN-test-economy`, a behavior the diff adds with no coverage).
**Fix:** Add a nil-`Index` row to an existing `BeginBatch` error-path test in `beginbatch_test.go`, asserting the error names the missing wiring and nothing panics.

### [BLOCKING] A per-helper test covered by the guard

**Location:** `cmd/lyx/treesitterlink_test.go:151`.
**Issue:** `TestParseGoListImports` pins the unexported `parseGoListImports`, which `TestTreeSitterLink_OnlyAllowedTestBinariesLinkIt` already exercises over real `go list` output, with a floor on the parsed test binaries that fails when the parse breaks; card 5 prescribes only the chain-search and comparison table test.
**Fix:** Delete `TestParseGoListImports`, which lowers the `cmd/lyx` count the budget row must follow (F7).
