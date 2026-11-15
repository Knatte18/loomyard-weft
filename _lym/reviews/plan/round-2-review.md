---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/plan/11-dobby-pass-act.md
    summary: >-
      Card 11 drops the CancelMarker check from planPass and drivePasses, so the cancel lock held before the intent is
      written, which card 8 makes readCancelMarker report for DobbyRuns, no longer stops a pass.
    fix: >-
      In card 11's Intent, change "the act rule takes the place of its `CancelMarker` and `Held` checks" so the act rule
      replaces only the `Held` check, and keep `run.CancelMarker` in `planPass`'s first gate beside `drivable` and in
      `drivePasses`' pending-lift drop condition (`!drivable(run) || run.CancelMarker || verdict is not act`).
      Keep the `RunView.CancelMarker` doc as the card words it.
      In `TestPlanPass` keep a row where `CancelMarker` is true and the verdict is act, giving no pass on every trigger.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lym/plan/05-pairteardown-read-run.md
    summary: >-
      Card 5 leaves PastParentMerge undefined for a run with no pair here, whose child record it reads from origin,
      since both inputs of battenshed.FinalizePastMerge need the task worktree.
    fix: >-
      In card 5's Inputs list, state that `PastParentMerge` is false, with neither seam called, when no pair stands here
      and the child was read through `fabricengine.OriginRunStatus`; the act rule hands such a run over before the input matters.
      Name the source of the `restepped` argument for a pair that stands: `sheddriver.ReadSettleRecords` over
      `shedrun.SettleRecords(task, shedrun.SelfRunID)` then `sheddriver.RestepRecorded`, as `childRestepped` does today.
      Add a `TestReadRun` row: a child at Finalize, running, read from origin with no pair here, reads `PastParentMerge`
      false with no error and calls neither seam.
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/plan/05-pairteardown-read-run.md
    summary: >-
      Card 5 plans a tier1 hubforge test in internal/pairteardown without naming its test package, but hubforge imports
      pairteardown, so an internal-package file importing it is an import cycle.
    fix: >-
      In card 5's paragraph on `TestReadRun_ReadsTheSameStateWithoutTheCache`, say it sits in the external test package
      `pairteardown_test`, as `teardown_integration_test.go` does, and calls the exported `pairteardown.ReadRun`.
      Say `TestReadRun` is in the internal package, since it drives the unexported `runReader`.
  - id: F4
    severity: LOW
    class: scope
    location: _lym/plan/05-pairteardown-read-run.md
    summary: >-
      Card 5 edits the const block of landRow and worktreeTeardownRow but lists only the file followup.go as its Edit
      target, so its Intent names symbols the targets give no member glyph for.
    fix: >-
      In card 5's `**Edit:**` section replace the line `internal/pairteardown/followup.go` with the two lines
      `internal/pairteardown#landRow` and `internal/pairteardown#worktreeTeardownRow`.
      A `lym loom validate-plan --overlay` run with this change reports no blocking finding the baseline lacks.
  - id: F5
    severity: LOW
    class: design
    location: _lym/plan/03-runfsm.md
    summary: >-
      The Landed refusal names lym board list to check it, a read that lifts nothing, so a landed slug whose pair still
      stands, or one a verb is re-run for, has no way forward in the text.
    fix: >-
      In card 3's `Verdict.Refusal` paragraph reword the terminal `Landed` clause to a way forward that lifts the cause:
      the slug has landed, remove its pair with `lym fabric remove <slug>` if one stands, then release the entry with
      `lym board set-status '{"slug":"<slug>","status":null}'` and delete the leftover task branch, or give the new task another slug,
      as the done-slug refusal of `lym batten run` words it.
      Update the `TestAct` terminal `Landed` row's expected text to match.
---

### [MEDIUM] Card 11 removes the only pass-planning check that sees the cancel lock

**Location:** `_lym/plan/11-dobby-pass-act.md`, with `_lym/plan/08-cancel-readers-intent.md` and `_lym/plan/07-cancel-intent.md`.

**Issue:** Card 7 writes the cancel intent only after the cancel lock is taken and the Windows live-process check passes.
Card 8 closes that window for the start guard, the pre-drive check and `DobbyRuns` by making `readCancelMarker` report a held cancel lock as present and live.
`DobbyRuns` carries that reading in `RunView.CancelMarker`.
Card 11 then says "the act rule takes the place of its `CancelMarker` and `Held` checks" and gives no pass unless the verdict is act.
`runfsm.Judge` reads the batten record alone, so in the window the verdict is act, `CancelMarker` is true, and nothing in `planPass` or `drivePasses` looks at it.
Today `planPass` returns none on `run.CancelMarker` (`internal/dobbyengine/passes.go:124`) and `drivePasses` drops the lift on it (line 203).
The spawned pass is refused by the start guard that card 8 fixed, so the harm is a refused pass and its notice, not a run driven past a cancel; the two cards still disagree on who sees the window.
`internal/dobbyengine/ended.go`, `runnotice.go` and `waitreminder.go` keep reading `CancelMarker`, so the field stays and only the planner stops using it.
The focus question is settled: a cancel is visible to the start guard, the pre-drive check and `DobbyRuns` from the lock take; it is not visible to the dobby's planner once card 11 lands.
After the intent is written, every reader sees it, because `Derive` reads the record on disk and the intent needs no commit to be read here.

### [MEDIUM] Card 5 does not say what ReadRun reads for a run with no pair here

**Location:** `_lym/plan/05-pairteardown-read-run.md`.

**Issue:** The card reads the child record "from the task worktree when the pair stands here, else `fabricengine.OriginRunStatus`", then derives `PastParentMerge` from `battenshed.FinalizePastMerge` over that record.
`FinalizePastMerge` calls its `restepped` seam and its `parentHolds` seam.
Today's `childRestepped` locates the task worktree and reads its settle records, and `parentHoldsTaskWork` (`internal/fabricengine/replay.go:308`) opens the pair at `pairLocation` and reads its HEAD (`Open(pairLocation)`, `CurrentSHA`).
Neither works with no pair here, and the card names no source for `restepped` at all.
`OriginParentHoldsTaskWork` is described as `ParentHoldsTaskWork` against `refs/remotes/origin/...`, so it inherits that need for the task HEAD.
A child read from origin at `Finalize`, running or transiently failed, would make `ReadRun` return an error, and every verb that asks the rule for that slug, `lym batten cancel` included, would refuse on it.
The act rule hands a run owned elsewhere over before `PastParentMerge` is read, so for a run this machine does not own the input is moot, and the fix states exactly that.
`go list -deps` shows `battenshed` does not reach `pairteardown`, so the new import closes no cycle.

### [LOW] Card 5's tier1 test needs the external test package

**Location:** `_lym/plan/05-pairteardown-read-run.md`.

**Issue:** `go list -deps ./internal/hubforge` lists `internal/pairteardown`, and the existing hubforge tests in `internal/pairteardown` are in `package pairteardown_test`.
`TestReadRun_ReadsTheSameStateWithoutTheCache` is "over a `hubforge` hub", and `TestReadRun` drives the unexported `runReader`, so the two cannot share a test package.
The card names neither, and an implementer who puts both in one file hits an import cycle in the test build.

### [LOW] Card 5's Edit target is a file path while its Intent names symbols

**Location:** `_lym/plan/05-pairteardown-read-run.md`.

**Issue:** The only `**Edit:**` target is `internal/pairteardown/followup.go`, a non-test Go file, and the Intent edits the doc comment of the const block holding `landRow` and `worktreeTeardownRow`.
The rubric asks for the member glyphs in that case.
The overlay run with the two glyphs replacing the file target reports `ok` with no blocking finding beyond the baseline's, so the fix passes `redundant-file-target`.

### [LOW] The Landed refusal offers a check, not a way forward

**Location:** `_lym/plan/03-runfsm.md`.

**Issue:** For a pause from the record's intent with no local pause entry, the wording is sound: `lym tender resume <slug>` runs `Verdict` with `Pause` set aside, then `Clear`, which runs before the lift, and `tenderengine.Resume` succeeds when no entry stands (`internal/tenderengine/resume.go:121-125`), so the intent clears.
For a landed slug the terminal wording names `lym board list` "to check it".
`lym tender resume` is covered, since card 14 treats `ReasonTerminal` as act there.
`lym batten run`, `lym batten step` and `lym loom resume` all return this text for a `Landed` slug with no batten record (board `done`), and a read-only listing lifts nothing.
When the record stands, card 10 returns the done-slug refusal that names the real path; the no-record case has none.

## Rejected

- The Shared Decision `verbs-that-do-not-ask` (`lym batten goto`, `lym loom run`, `lym loom step`) is not raised again.
  Round 1 raised the same exemption as its F15, offered stating the reason and its bound as an accepted fix, and the plan states both.
  The asides the focus names are licensed: the shed-pause aside by the parent's entry "A shed pause is an aside of the act rule", which names the same six verbs; the pause, cancel and hold asides by D1's text on `lym tender resume` and `lym batten cancel`; the owed-push aside by D3's text that those verbs act at once offline and owe their push.
- Card 5 does not build a gate it cannot pass.
  `internal/pairteardown/teardown_test.go:15` already imports `battenrecipe` in the internal package, and `go list -deps ./internal/battenrecipe` shows no path to `pairteardown`, so the tier0 test's `battenrecipe`, `contracts/recipes` and `shedbuild` imports close no cycle.
  `followup.go` keeps `landRow` and `worktreeTeardownRow` local, and `runShedRow` joins them there as the card says.
  The tier1 test is the only part that needs care (the finding on the test package).
- Card 11's import set is complete: `RunView` gains `runfsm.State` and `runfsm.Verdict`, `dobbyengine` imports only `runfsm` anew, and the card adds that path to `dobbyengineAllowedImports`, which `TestToldGeometryInvariant_AllowlistOnly` reads.
  `drivable` reading only running leaves no clause the rule makes unreachable; `RunView.ReachedParent` stays for `strandedEnded`.
  `DobbyRuns` keeps `ReachedParent` and `CancelMarker`, both still read.
- No read `ReadRun` makes per listed run runs `git ls-remote` or another network call.
  `FetchedTaskBranchOnOrigin` reads in-process refs as `TaskBranchSlugs` does, `OriginRunStatus` reads the fetched records ref the way `UpstreamRunUnfinished` does, `OriginParentHoldsTaskWork` is a local merge-base against a remote-tracking ref, and the board, Bolt owners, pause record and settle records are local files.
  `Topology.TaskBranchOnOrigin`, which does run `ls-remote`, is used only by `lym batten cleanup` (card 9), never per tick.
- `PATTERN-test-only-code`: every new function on cards 2 to 5, 13, 15 and 16 has a production caller on a card of this plan, and `Order` is reached through `Derive`.
  `PATTERN-leaf-packages`: card 3 extends the entry for `runfsm`.
  `PATTERN-fabric-boundary`: card 4's three reads sit in `internal/fabricengine` and learn nothing of the board.
  Card order holds: each card's `Uses` name an earlier card, and the shared edits of `endRunPhase`, `clearRunPhase`, `hubCancelDeps` and `Teardown` run cards 7, 8, 9 and 13 in that order.
