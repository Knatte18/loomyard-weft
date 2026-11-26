---
dispositions:
  - id: F8
    disposition: fixed
---

# Round 2 fixer report

## Changes

- F8: the range key is now the sha256 of the range's full committed diff, so a file the range adds counts.
  - New seam `Deps.RangeDiff`, wired in `internal/loomcli/driftwiring.go` to the existing `Fabric.ReadCommittedDiff`.
    No new `fabricengine` method was needed.
  - `listFiles` builds `rangeKey` from that diff; the judge's listed files are unchanged.
  - The seam is required in `internal/driftengine/verify.go` and `internal/shedrecipe/entries_drift.go`,
    and added to the `shedrecipe`, `envkit` and `battencli` seam fixtures.
  - The comments in `loosened.go`, `records.go`, `verify.go` and `doc.go` state the new coverage.
  - Tests in `verify_test.go`: a new row for dropped checks added in a new test file.
    The `carried_by` row's carrier is now a file the range adds.
- Gate: `lym gate test` over `driftengine`, `shedrecipe`, `battencli`, `testkit/envkit` and `loomcli` passed in tier0.
  After the self-check change, `driftengine` passed again.
- Gate findings: two comments in `internal/driftengine/loosened.go`, on `rangeRulingKey` and `listFiles`, wrapped mid-sentence.
  Both are rewritten as one sentence per line, and `driftengine` passed again.

A ruling stored under the old key that holds an uncaused check or names a carrying file is judged once more.

`docs/overview.md` and `_lym/landing/summary.md` needed no change: what they say about the re-judge now holds.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, fixed as proposed: the no-review-base case now also asserts that `RangeDiff` is not read.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
