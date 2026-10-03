---
verdict: APPROVED
rationale: "All five round-2 findings are fixed in the cards and hold against the tree: card 10 converts internal/gitrepo/noforceadd_test.go (floor 5, banned tokens kept), card 15 edits names.go's loomRowEngines comment, card 11 edits seamsignature_test.go and sandbox_coverage_test.go, card 28 keeps loomcli's commitStatusDeps and newCommitStatusSeam(deps) for commitrecords.go and the protected integration test, and card 27 rewrites shedverbs' mirroring comment; a sweep for further removed names and unconverted invariant scans found nothing new"
---

# Round 2 judgment

## The round-2 report

The report has no `## Focus departures` section, so there are no departures to ratify or reject.
Each of its five findings is legitimate under the rubric.
None restates a mechanical check, and none flags a missing dependency edge or a `Rename` card's `ImpactSummary`.

- F1 (BLOCKING) is a fidelity gap against D2.
  `internal/gitrepo/noforceadd_test.go`'s file comment says it machine-checks the Never Force-Add Invariant, and it walks source.
- F2 and F3 are target-list gaps.
  A card removes a name, and a file outside that card's targets still names it.
- F4 is an Intent gap.
  Card 28 said the CLIs "lose their private copies", while untargeted `commitrecords.go`, `commitrecords_test.go` and the protected integration test use `loomcli`'s `commitStatusDeps` and `newCommitStatusSeam(deps)`.
- F5 is a dangling comment inside a card 27 target, which the Intent did not cover.

## Fixes checked against the plan and the tree

- F1: card 10 targets `internal/gitrepo/noforceadd_test.go`.
  Its Intent converts the scan to a shallow production-file walk of its own package directory, keeping `noForceAddBannedTokens` and the floor of 5, with one planted `"-f"` as proof.
  The tree has `noForceAddMinScannedFiles = 5`, so the stated floor matches.
- F2: card 15 targets `internal/loomrecipe/names.go`, the source of the comment at lines 16–18.
  It describes the edit as comment-only, and the ImpactSummary names the one production doc comment.
- F3: card 11 targets `cmd/lyx/seamsignature_test.go` and `cmd/lyx/sandbox_coverage_test.go`.
  Their comments name `clitree_test.go`'s walk instead of the deleted files.
- F4: card 28 keeps `loomcli`'s `commitStatusDeps` with its flat field set and its `newCommitStatusSeam(deps)` signature as a wrapper.
  It names the three dependent files as the reason.
  `commitRecordsVerb` in the tree calls only `d.MergeActive`, `d.Commit` and `d.Push` on that type, so leaving the type in place is enough.
  `fabricengine/pushanchored.go:35` names `loomcli`'s `newCommitStatusSeam`, which stays.
- F5: card 27 rewrites `shedverbs` test comments that cite a deleted `loomcli` test.
  It quotes the `step_test.go:24` comment as its example.

## Sweep with no new finding

- Removed and renamed names, outside `_lyx/`:
  - Card 14's deleted `constructoranchoring_test.go` and `notransients_test.go` are named only by each other.
  - Card 16's `recognizedFormat` is a `Rename`, so its in-package reader `containment_test.go:136` is part of the rename.
  - Every test that cards 12, 15, 27 and 28 delete is named only in its own doc comment.
  - No `.md` file outside `_lyx/` names any of them.
- Unconverted invariant scans:
  - I listed every `_test.go` that reads source (`os.Getwd`, `os.ReadDir`, `filepath.Glob`/`Walk`, `parser.Parse*`, `runtime.Caller`) and mentions an invariant, then kept the ones no card targets.
    Most of those read runtime fixture directories, save the cwd before a `chdir`, or are the smoke tests card 21 already discusses.
  - `shuttlecli/modelpin_test.go` is an R2-F3 regression guard and enforces no invariant.
  - `burlercli`/`webstercli` `wiring_test.go`'s watchdog scans cite a batch scope note.
  - `shuttleengine/run_test.go`'s `TestShuttleengine_LiveSubstrateLoggingGoesThroughLogger` does read its package's sources.
    It presents itself as R2-F8's regression guard and mentions an invariant only as one it satisfies (Test Tier Purity), not one it enforces.
    `CONSTRAINTS.md` lists it nowhere as an enforcer.
    That matches the excluded `shuttlecli/modelpin_test.go`, so D2 does not reach it.
    As a regression guard citing a finding, D21 protects it, and no card touches it.
