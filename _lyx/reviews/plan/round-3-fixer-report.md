# Fixer report — plan, round 3

## Changes

- **F1** (BLOCKING, fixed as proposed): card 17 creates `readingWaitingHeld` (`"waiting_held"`) and edits `checkCorpusParse`, `corpusReplay.checkRead` and the `master-fork-running` `case.yaml`.
  The Tests part states that the case's last turn end is now held at once with one notice, reads `waiting_held` (parsed like `waiting`, checked for one notification), and still ends `OutcomeTimeout`.
  It also states that `what_went_wrong` and the last `old_reading` are reworded to D11, while the fork turn ends and `states` stay as they are.
  The ImpactSummary names the corpus case.
- **F2** (MEDIUM, fixed as proposed): card 11 builds the stopped refusal's way forward from `report.Branches`, never `EntryOutcome.WayForward`, skipping the slug's gone branches, with the reason taken from `classifyEntries`' ordering.
  `TestRemoveVerdict` gains two gone-first rows: gone code then diverged records, and gone code then diverged Bolt.
  Each asserts the other branch's way forward and no `lyx fabric remove` step.
- **F3** (MEDIUM, fixed as proposed): card 2's `TestEnd` gains the rows for a descendant that leads its own group.
  When that group has no live member, or only an earlier-started one, the descendant gets no group signal at the SIGTERM, and the same holds between the SIGTERM and the SIGKILL.
- **F4** (MEDIUM, fixed as proposed for card 10, fixed differently for card 12):
  - **Card 10.** `pausedIn` returns `covered, hubEntry`, and `hubEntry` is threaded through `awaitOrPass`, `awaitRunLockOrPass` and `runLockRefusal`.
    The paused refusal names `lyx tender resume` with no slug whenever a hub entry covers the slug, and `<slug>` only when slug entries alone cover it.
    `TestRunLockHolderPasses` gains a hub-only refusal row and a hub-plus-slug refusal row.
    The `TestTeardown_CallSequence` pause rows assert the `hubEntry` the fake saw.
  - **Card 12, fixed differently.** `EndBeforeSessionDown` passes both readings.
    The proposal had its `TestEndBeforeSessionDown` hub-pause row assert the refusal text.
    Instead, the covering-pause row is split into a slug pause and a hub pause that assert the `hubEntry` the fake `awaitOrPass` saw.
    That test fakes `awaitOrPass`, so the refusal text there is the fake's own and would pin nothing; `TestRunLockHolderPasses` already pins the real wording.
- **F5** (LOW, fixed as proposed, the review's substrate alternative):
  - **The change.** `busyError` reads owned processes through a new `ownedProcesses` substrate, wired in `New` to `runproc.List(fabricengine.RunProcDir(hub), slug)` and faked in `fakeSubstrates.teardown`.
    A read error is named in the refusal.
    `TestProbeQuietSeesTheGoDriverLock` is added to **Edit:** and sets the field.
    `RunProcDir` is added to **Uses:**, and the ImpactSummary is updated.
  - **Why not the direct read.** The review's first option, a direct `runproc.List` read, was not taken.
    `fakeSubstrates.teardown` uses the fake hub `/hub` (`internal/pairteardown/teardown_test.go:50`), so card 10's busy-driver row could not script the owned processes it asserts.
    Card 2 also states no empty-list behaviour for an absent record directory, so the review's claim that the direct read returns an empty list has no support in the plan.

## Self-check

- fix-consistency, MEDIUM: card 11's way-forward rule could pick a `BranchBusy` branch the gone-branch exception leaves out.
  This duplicates the fix-proposal fork's first finding.
  Fixed as proposed: the rule leaves out the slug's own records branch and any shared branch reading `BranchBusy`, and `TestRemoveVerdict` gains the row gone code, busy own records, diverged Bolt.
- fix-proposal, MEDIUM: card 11 busy-branch selection, the same finding as the fix-consistency one above.
- fix-proposal, MEDIUM: the no-slug `lyx tender resume` lifts every live entry, including other pairs' pauses and a pause held for another machine (`internal/tenderengine/resume.go:20`, `:106`).
  Fixed with my own change.
  The proposed `lyx resume` was not taken, because `tendercli` documents it as a shortcut for `lyx tender resume`, which lifts the same set, and the engine has no hub-only lift.
  Card 10's refusal instead says that the no-slug form lifts every live entry of the hub, and the hub refusal rows assert that.
- fix-claims: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
