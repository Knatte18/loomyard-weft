---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "Not done: one set of caveats per linking package. The tests of all linking packages share one caveat per cause, each test named `<package>.<Test>`. The note is cut at reachNoteLineCap (40) lines (internal/gatecli/reachreport.go, NoteLines), so per-package caveats for a widely linked package would push the later unreached functions, unrefreshed lines and errors out of the note."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

## What changed

- F1: `staleLinked` rebuilds a worktree map of an older format. The reach integration test has an `oldformat` package whose map is fresh by hashes at version 2.
- F2: `testreach.Reader.UnseenLinkerTests`, and its caveats in `reachStep.caveats`. Pinned by a `TestReader` row and the integration test's `dirtylink.One` caveats.
- F3: the reach integration test compares exact caveat lists in place of the `Caveated` boolean.
- F4: `unseenCause` names an unresolved test that covered nothing as unresolved. `TestUnseenTestCaveats` has that row and a five-test row.
- F5: `unmappedTierNames` takes the declared mapped tiers and names tests with no tier. Pinned by `TestUnmappedTierNames`.
- F6: a `TestReaderIndex` row with a current stamp and version 1. It also drops the `unseen` key, since a row that only changed the version would pass with the version check removed.
- F7: the `judgeRuns` doc line.
- F8: `_lym/landing/summary.md` states what an older binary does with the new shared maps and index. The blast radius stays `gate`, with the affected surfaces named.
- Docs and `_lym/landing/summary.md` follow F1, F2 and F5.

Commits: F7 alone; F1, F2, F3, F4, F5, F6 and F8 together; one each for SC1, SC3 and SC4 below; then one for the gate's comment findings.

After the round, the gate reported two comment lines this round added: an over-long one on `unmappedTierNames` and a wrapped break in `testreach/doc.go`.
Both are rewritten as shorter sentences, with no code change.

Verification: `lym gate test --tier 0 --tier 1` over `internal/gatecli`, `internal/testreach` and `internal/testcover` passed after the review fixes and again after the self-check fixes.
The comment repair was checked with `go vet` on `internal/gatecli` and `internal/testreach`.
The new tests were not run against the unfixed code.

A later gate run printed a reach report naming functions no mapped test reaches: `reachStep.caveats`, `reachStep.staleLinked`, `runReach`, `fillMapTests`, `testMapRun.mapPackage` and `testMapRun.runTest`.
No test was added and no code changed for it.
Tests that drive these paths exist and passed in the tier1 run after the self-check fixes: `TestGateTestReachStep` (the caveats and the refresh), `TestGateReach` (the verb) and `TestGateTestMap` with `TestGateTestMapLinkedPackages` (the map run).
The report's caveat text is the wording this branch replaces, so the `lym` that printed it predates the change.
I infer that its map run entered these tests unrun as unresolved, which is the defect this branch fixes; I did not read the maps to confirm it.

## Self-check

All four lenses returned a report. Fork findings that name one defect share an SC id.

- SC1, fix-code-correctness, BLOCKING: `lym gate reach` built its step with no gate config, so F5 made it call every tier unmapped. Fixed.
- SC1, fix-consistency, BLOCKING: the same defect. Fixed.
- SC1, fix-proposal, BLOCKING: the same defect. Fixed.
- SC1, fix-affected-set, BLOCKING: the same defect. Fixed; the `TestGateReach` row asserts that no unmapped-tier line appears.
- SC2, fix-consistency, LOW: the summary's Tests section was stale. Fixed.
- SC3, fix-consistency, LOW: the mutate docs said matching file hashes suffice. Fixed in `gatecli/doc.go` and the verb's help.
- SC4, fix-affected-set, LOW: an older-format worktree map that matched the files still overrode a later shared map. Fixed: `worktreeMapWins` requires the current format for the matching-files rule, through the new `testcover.Map.HasCurrentFormat`.

## Disputed

None.

## Deferred

None.

## Rulings

None.
