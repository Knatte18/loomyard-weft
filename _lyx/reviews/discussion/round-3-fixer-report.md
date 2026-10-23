# Fixer report — discussion, round 3

Every finding's premise held against the code; all six are fixed in `_lyx/discussion/decision-record.md`, with the D7 rationale, three ledger picks and a round-3 entry in `_lyx/discussion/support-log.md`.

## Changes

- F1 (MEDIUM, design): fixed differently.
  D10 now names batten's owner-pause arm of `lockStep` and `ownedElsewhere`, their `keepsLockForFinalize` and `FinalizePastMerge` read, and the choice: `FinalizePastMerge` is unchanged and those arms keep releasing, since a run another machine owns must not land here.
  Before step 5 Finalize re-reads the lock record through its parent guard.
  Not made as proposed: taking the lock again and re-running steps 2 to 4 when the record no longer names the run.
  With the idle paths guarded, only an owner pause or another machine's ownership releases a running Finalize, and the owner-pause arm fires on a live pause record (`internal/pauserecord/record.go:84`, `OwnerHeld`), which step 1's owner checks do not read, so re-taking would land during the pause or ping-pong with batten's next probe.
  Finalize instead halts, landing nothing, with `lyx loom resume` re-running it from step 1 under a lock taken afresh; an unreadable record halts as at the guard's acquire.
  The re-read is not atomic with the landing, which D10 and Open risks now state.
  An acceptance criterion covers the halt and the owner-check stop.
- F2 (MEDIUM, decision): fixed as proposed.
  D2 picks a `MergeInAt(source, warpSHA)` fabric method, threads a warp SHA through `mergeresolve.Resolve` and its `Fabric` seam, and has Finalize pass none.
  Scope's In list gains `mergeresolve`'s interface change and its Out line says the session's behavior is unchanged while its entry seam gains the SHA; the local-ahead criterion names its test surface.
- F3 (MEDIUM, design): fixed as proposed, placed differently.
  D11 adds a `hand_back` field on the stop envelope and `lyx shed status`, and edits the stencil's Re-launching bullet, Parking step 3 and the guide's Binary watch to skip a marked stop.
  The predicate is a hand-back check told to `shedverbs`' Spec by the recipe's owner, as `GotoGuard` is (`internal/shedverbs/doc.go:16`), rather than a test naming Merge-In and Finalize inside `shedverbs`, which stays recipe-generic; batten's re-step eligibility reads the same predicate.
  Scope and the changed-build criterion follow.
- F4 (MEDIUM, decision): fixed as proposed, picking the union of every declared language's patterns for darn.
  D10 step 4 reads the loom plan's `language:` (absent as `go`), the union for darn, and every file as a test file for `none` or a language the table lacks, and states the default differs from `sourceExtensions`.
  Acceptance rows added for a darn C# `FooTests.cs` fix and a `language: none` small fix.
- F5 (MEDIUM, design): fixed with the first option.
  D4 binds the `merge-in` fixer's change by D10's size and kind caps when the Code-Review budget is 0, halting with the landing-stop prefix; D13's bound sentence names both fixers; an acceptance criterion added.
- F6 (LOW, design): fixed as proposed.
  D7 and the support log's ledger line state that a `Command` row would pass on as the stub does, that `PATTERN-command-row` was weighed, and that the row is the seat of the human PR review engine and must not take an operator shell command.

## Self-check

- fix-consistency, LOW, "the two above" count stale after F5's new halt in D4: fixed, "those above".
- fix-consistency, LOW, Scope and D10's language rule name only Finalize: fixed, both now cover Merge-In's budget-0 cap.
- fix-consistency, LOW, hand-back criterion omits the Merge-In over-cap halt: fixed, added.
- fix-consistency, LOW, batten's re-step check a third copy of the landing-halt test: fixed, it reads the `hand_back` predicate.
- fix-consistency, LOW, "absent language" in D10 read as the absent key: fixed, "a language the table lacks".
- fix-claims, MEDIUM, `MergeInAt` naming `source` for the merge message: fixed, git merges the SHA (`internal/fabricengine/merge.go:260`), so the text says the merge state's `Source`.
- fix-claims, MEDIUM, and fix-way-forward, MEDIUM (merged), the budget-0 Merge-In over-cap halt recurs on resume and has no exit once attempts are spent: fixed, the cap measures from the HEAD at the row's start, a resume included, and the resumed Merge-In after that halt reads the operator's review as the alignment, advancing the aligned tip over a passing gate with no fixer.
- fix-way-forward, LOW, the re-check names no outcome for an unreadable record: fixed, Stuck with Halt as at the guard's acquire.
- fix-proposal, MEDIUM, F1's re-check races the landing and an owner pause is not ownership: fixed differently, see F1 above; the race is stated in D10 and Open risks.
- fix-proposal, LOW, `none` plus F5's budget-0 cap halts nearly every such run: fixed, stated in Open risks.
- fix-proposal, LOW, F3's predicate puts row names in recipe-generic `shedverbs`: fixed, a told Spec check.

## Disputed

None.

## Deferred

None.
