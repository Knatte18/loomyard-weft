# validator-checks: discussion, seam and card-check validators

All three batches reached `done`; every record-batch digest returned `ok` with no warnings.

## 01-decision-id-check (head fbbe1de3)

Cards 1 to 4: decision ids and acceptance coverage in both discussion checks, citation extraction in the record parser, `ResolveGlyphs` on the plan index, and cited-path and cited-symbol checks on a loom record.
Scope guard deviations: `internal/discussionparser/citations.go`, `internal/discussionparser/decisionids.go`, `internal/loomshed/citations.go`, `internal/loomshed/gates.go`, `internal/planglyph/index.go`, `internal/planindex/planindex.go`.
Departure: card 4's integration test builds its repo with `git init` through `gitkit.Git` instead of `gitkit.CopyRepo`, which a guard pins to `internal/lymcwd` (`PATTERN-leaf-packages`); recorded as a `Pattern-Departure:` trailer in the card 4 commit.
The digest listed no renames.

## 05-seam-marker-parser (head d7fce7ac)

Cards 5 and 6: `//lym:seam` parsed beside `//lym:guard` in either order, the PATTERN line and background updated, and a tier0 scan that refuses an unmarked reflective test, with the existing reflective tests marked.
Scope guard deviations: `internal/impactset/seams.go`, `internal/testkit/envkit/envkit_test.go`.
No departures or renames.

## 07-test-tier-check (head 8a7208e1)

Cards 7 to 9: the `test-tier-unstated`, `seam-test-unlisted` and `doc-go-unlisted` plan checks, with `TestValidatePlan_CardChecks` carrying their rows.
The parent decision on tier tags applies: a file's tags are collected as a set.
Scope guard deviations: `internal/loomcli/parity_test.go`, `internal/loomshed/docgo.go`, `internal/loomshed/seamtest.go`, `internal/loomshed/testtier.go`, `internal/shedrecipe/fixture_test.go`.
No departures or renames.

## Verify gate findings

The plan-level verify failed on attempt 1 of 2 in package `github.com/Knatte18/loomyard/cmd/lym`; the findings file recorded no card touching a failing package.
A fixer fork was spawned for it.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `06e6a5168949879b9471c83e4e73ef9a614fb7ca` fix: bring validate-plan's Long page under the byte cap and gofmt deleteorder_test.go
