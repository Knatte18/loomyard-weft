---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/17-shuttle-shell-only-waiting.md:47
    summary: "Card 17 leaves the claudeengine corpus replay of master-fork-running failing: its last turn end (gated, outputs missing, one non-awaited shell) is now held and notified, but the case expects reading waiting with zero notices"
    fix: "In card 17 add to **Edit:** the targets `internal/shuttleengine/claudeengine_test#checkCorpusParse`, `internal/shuttleengine/claudeengine_test#corpusReplay.checkRead` and `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running/case.yaml`, and to **Create:** `plan:internal/shuttleengine/claudeengine_test#readingWaitingHeld` -> `const readingWaitingHeld = \"waiting_held\"` (this exact target set passed `lyx loom validate-plan --overlay` with the baseline ok). In the Tests part of the Intent state: the case's last turn end (gated run, output file missing, one payload-reported shell, no awaited prefix in the replay's Spec) is now judged as a plain turn end with an output missing, so it is held at once and the notifier is called once; its `reading` becomes `waiting_held`, `checkCorpusParse` reads `waiting_held` like `waiting` (an EventWaiting carrying the case's tasks), `checkRead` expects exactly one notification for it, and the run still ends OutcomeTimeout at the deadline; the `what_went_wrong` text ('stays waiting on the shell whether or not Master's output files exist; only the run deadline, the liveness check or a stop ends such a run') and the last turn end's `old_reading` text are reworded to D11. The four fork turn ends and the `states` list stay as they are."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-fabric-remove-reconcile.md:36
    summary: "Card 11 refuses a stopped verdict with the entry's way forward, but the entry's way forward is the first stopping branch's, which for a gone code branch beside another stop is the gone branch's text naming lyx fabric remove itself, so the refusal loops"
    fix: "In card 11's Intent, say that `removeVerdict` builds the refusal's way forward from `report.Branches`, never from `EntryOutcome.WayForward`: it takes the `WayForward` of the first stopping branch that is not a `BranchGone` branch of the slug (a gone branch's `stopWayForward` text, `FailedPushWayForward` of class gone, says that `lyx fabric remove` of the pair removes it), whether that branch is the slug's own (diverged, dirty, rewound, rewritten) or a shared Bolt or prime-records stop. Reason: `classifyEntries` sets the entry's way forward to the first stopping branch of the slug in target order, which puts the code branch before its records sibling, and keeps the slug's own stop ahead of the shared D3 stop; so a gone code branch beside a diverged records branch, or beside a Bolt stop, gives an entry way forward that names the refused verb. Add `TestRemoveVerdict` rows with the gone branch listed first: gone code branch then diverged records branch, and gone code branch then a diverged Bolt branch, each refusing with the other branch's way forward and not containing `lyx fabric remove` as the step. The other stopping states (rewound, rewritten, diverged, dirty) already give steps other than re-running the removal, and busy and live never stop the run scope's own slug."
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-runproc-owner-record.md:74
    summary: "Card 2 names no TestEnd row for the group identity check of a descendant that leads its own group, in either the SIGTERM or the SIGKILL phase, though the Intent applies the group check to it"
    fix: "In card 2's TestEnd rows add two rows over the fake process table: a read descendant that leads its own group (a push's remote git child) whose group no live process reports any more, or reports only through a process with a start time before the one read with the descendant, gets no group signal at SIGTERM; and the same state arising between the SIGTERM and the SIGKILL gets no SIGKILL group signal. The existing rows for a recorded leader's group (no live member, member with an earlier start time, same cases between the phases) cover the recorded leader's check only; the descendant's group check is a separate branch ('and, for its group, the group check above'), so a positive row alone does not reach its guard."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-teardown-skip-readings.md:63
    summary: "Card 10's paused refusal names lyx tender resume with no slug when a hub entry covers the run, but runLockRefusal, awaitOrPass and pausedIn carry only a paused bool, so no input selects that form and no test row reaches it"
    fix: "In card 10 make `pausedIn` report whether a hub entry covers the slug besides whether any live entry does (return `covered, hubEntry bool` and an error), thread `hubEntry` through the `awaitOrPass` substrate (`func(ctx context.Context, slug string, paused, hubEntry bool) error`), through `awaitRunLockOrPass` and into `runLockRefusal(holder int, holderKnown bool, slug, lockPath, goos string, paused, hubEntry, parentLocked bool)`; the paused-with-parent-lock refusal names `lyx tender resume` with no slug whenever a hub entry covers the slug, also when a slug entry covers it too, and `lyx tender resume <slug>` only when slug entries alone cover it. Reason: `lyx tender resume <slug>` is refused for a slug that only a hub entry covers (tendercli doc: the engine refuses a slug that only a hub entry covers, and the hub pause is lifted by `lyx resume`), and beside a hub entry it would leave the pause standing, so the named step could not succeed. Add `TestRunLockHolderPasses` refusal rows for a hub-only entry and for a hub plus slug entry asserting the no-slug form, and update card 12 to pass the same flag when `EndBeforeSessionDown` calls `awaitOrPass`, with its `TestEndBeforeSessionDown` hub-pause row asserting the refusal text when the lock is held."
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/10-teardown-skip-readings.md:71
    summary: "Card 10 makes busyError list the slug's owned processes without saying how it reads them, and TestProbeQuietSeesTheGoDriverLock calls busyError on a bare Teardown literal with no substrates, so a seam-based read panics there"
    fix: "In card 10's Intent state that `busyError` reads the slug's owned processes with `runproc.List(fabricengine.RunProcDir(t.prime.HubPath), slug)` directly, which needs no substrate and returns an empty list for an absent record directory, so the integration test `TestProbeQuietSeesTheGoDriverLock` (`internal/pairteardown/probequiet_integration_test.go`, which builds `&Teardown{prime: ...}` and calls `busyError`) keeps passing unchanged; or, if a substrate field is used, add `internal/pairteardown#TestProbeQuietSeesTheGoDriverLock` to **Edit:** and state that it sets the field."
---

### [BLOCKING] Card 17 breaks the claudeengine corpus replay of master-fork-running

**Location:** `_lyx/plan/17-shuttle-shell-only-waiting.md:47`
**Issue:** The corpus case `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running/case.yaml` is `gated: true` and its last turn end reads `waiting` with `outputs_exist: false` and one payload-reported shell, no awaited prefix (the replay's `Spec` sets none).
`TestCorpus_ReplaysEachTurnEnd` (untagged, so card 17's tier-0 gate runs it) drives that through the real `Wait`.
Under card 17 a gated waiting turn end whose list is only non-awaited shells is judged as a plain turn end, and with the output missing it returns a held turn end, so `Wait` calls `notifyHeld` and the replay's notifier counts one notice.
`corpusReplay.checkRead` asserts `notified == 0` for a `waiting` reading and fails the run on the next `Sleep`.
The case's `what_went_wrong` and `old_reading` prose also state the old rule.
The card lists five tests in `internal/shuttleengine` and `attach_asking_test.go` but names neither this test file nor the case, so the card cannot pass its own gate.
Checked and not affected: `dialog_test.go` (its five-minute run never reaches the ten-minute bound), `waitshadow_test.go` (its shell rows either have the outputs present or advance no clock), `session_test.go` and `sessionstate_test.go` (session state, not the wait), and the `recoverbatch_test.go` rows in `internal/websterengine` (the recover-batch strand classification in `strand.go`, which does not use `ShellWaitBound`).

### [MEDIUM] Card 11's stopped refusal can name `lyx fabric remove` as its own way forward

**Location:** `_lyx/plan/11-fabric-remove-reconcile.md:36`
**Issue:** The card refuses `stopped` "with the entry's way forward, then `lyx fabric remove <slug>` again", and sends a stop whose only stopping branches of the slug are gone through today's request because a gone branch's way forward says `lyx fabric remove` removes the pair.
For a gone branch beside another stop it says the refusal names "that other branch's way forward".
`classifyEntries` (`internal/fabricengine/originreconcile.go`) fills `EntryOutcome.WayForward` with the first stopping branch of the slug in target order, the code branch before its records sibling, and the slug's own stop before the shared D3 stop.
A gone code branch beside a diverged records branch, or beside a diverged Bolt, therefore gives an entry way forward that is the gone text, so a refusal built from the entry names the refused verb and the operator loops.
The card does not say the text comes from `report.Branches`, and a `TestRemoveVerdict` fixture that lists the diverged branch first would pass an implementation that reads the entry.
For every other stopping state a run-scope reconcile produces for the slug (rewound, rewritten, diverged, dirty) the way forward is a step other than re-running the removal, and a busy or live stop never reaches the own slug in the run scope.
A dirty stop from a pair behind origin means `--force` cannot bypass it; the way forward (`git commit` or `git checkout --` in the worktree) can succeed, so this is a consequence of the decision record, not a plan defect.

### [MEDIUM] Card 2 has no TestEnd row for a descendant's group check

**Location:** `_lyx/plan/02-runproc-owner-record.md:74`
**Issue:** The Intent makes each read descendant's signal follow a fresh start-time comparison "and, for its group, the group check above", and repeats every start-time and group check before each SIGKILL.
The named rows cover: a recorded pid with a differing start time, a descendant with a differing or reused start time at each phase, a recorded leader whose group has no live member or only an earlier-started member, and the same two cases between the phases.
The only row that reaches a descendant's group is the positive one (a push's remote git child gets a group signal).
The guard that withholds a group signal from a descendant leading its own group has no named input at SIGTERM or at SIGKILL, so an implementation that skips it still passes `TestEnd`.
A descendant's group that is signalled after its leader exited, with a pid reused into the group id, is the case the guard exists for.
`endProcess` has its rows (differing start time, SIGTERM, SIGKILL after grace, reuse between phases, survivor) and is not part of this finding.

### [MEDIUM] Card 10's hub-pause resume form has no input and no test

**Location:** `_lyx/plan/10-teardown-skip-readings.md:63`
**Issue:** The paused-with-parent-lock refusal "names the way forward of resuming the run (`lyx tender resume <slug>`, or `lyx tender resume` when a hub entry covers it)".
The signatures carry one `paused bool`: `pausedIn` returns `(bool, error)`, `awaitOrPass` takes `paused bool`, and `runLockRefusal` takes `paused, parentLocked bool`.
Nothing tells the refusal whether a hub entry covers the slug, and the refusal rows name only `lyx tender resume slug`.
The branch that picks the no-slug form has no reaching input or test.
The form matters: `lyx tender resume <slug>` is refused for a slug only a hub entry covers (`internal/tendercli/doc.go`, `cli.go` resume help), and beside a hub entry it lifts the slug entry and leaves the hub pause, so the step named could not succeed for the hub-pause state that produces the refusal.
Card 12 passes the same `paused` flag to the same substrate, so it changes with the fix.
Every other input of the two functions is read by its rule: `holder` and `holderKnown` by the marker match (an unknown holder never matches), `waiting` and `slug` by the marker match, `paused` and `parentLocked` by the paused pass, and the paused pass is stated and tested for a known and an unknown holder.
The other refusal steps (`lyx tender pause <slug>` for an unpaused holder with no parent lock record, the retry for a step merging under the parent lock, the run lock path and `ps` or Task Manager for an unknown holder) can succeed for the state that produces them.

### [LOW] Card 10's busyError change meets a bare Teardown literal in an integration test

**Location:** `_lyx/plan/10-teardown-skip-readings.md:71`
**Issue:** The card has `busyError` add "the slug's live owned processes (role and pid)" and names `teardown_test.go:189` as the one test asserting its wording.
`TestProbeQuietSeesTheGoDriverLock` in `internal/pairteardown/probequiet_integration_test.go` builds `&Teardown{prime: ...}` with no substrates and calls `busyError("slug", st)`.
Card 9 reads owned processes through substrates wired in `New`; if `busyError` reads the list through such a field, that test calls a nil function.
The card names neither the read nor the test.

## Rejected

- Gated shell-only waiting turn ends and the display mark in card 17 are consistent: the mark can show only for an ungated run with shells at staggered first-seen times, as the card states, and `TestShellWaitMark` is rewritten for that state.
- The `attach_asking_test.go` row left unchanged in card 17 holds: the expired held turn end carries the waiting line's offset, which `NotifiedOffset` already records.
- Card 11 drops D5's "a reconcile that already removed the pair itself is read as done" with a reason that holds: `reconcileEnded` skips `scope.Run`, so a run-scope report never shows its own slug removed.
- `lens-review.md` in the plan directory was not read; it is not a card the Card Index names.
