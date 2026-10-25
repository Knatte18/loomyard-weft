---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/17-fabric-anchor-verb.md:44
    summary: "the unfinished-run refusal on a worktree the board names another machine as owner of loops: its way forward is lyx batten cancel, which refuses a slug owned elsewhere and names lyx adopt, which stops on the anchor_stopped weft and names the --set again"
    fix: >-
      In card 17, change the unfinished-run refusal of `changeAnchor` (the clause on line 44 and the matching `TestChangeAnchor` rows on line 62) so that for a worktree the board names another machine as owner of, the run status is judged from the weft's fetched upstream tip instead of the checked-out copy,
      through a new engine reader in card 15: add `Create: plan:internal/fabricengine#UpstreamRunUnfinished -> func UpstreamRunUnfinished(weftPath string) (unfinished bool, err error)`, which reads the run status files at the fetched upstream tip's tree with the tree-relative paths `shedrun.TreeRunFileRels` gives, and say so in card 15's Intent.
      The driver-lock and registered-batten-run refusals stay as they are for every worktree.
      When the upstream tip's status still reads unfinished, the refusal names `lyx batten cancel <slug>` run on the owner machine, which that owner accepts, and then the `--set` again; it no longer names a cancel on this machine.
      Add `TestChangeAnchor` rows: an owned-elsewhere worktree whose checked-out status reads unfinished and whose upstream tip's status reads finished is not refused; one whose upstream tip's status reads unfinished is refused naming the owner machine's cancel.
      If the author prefers to exempt an owned-elsewhere worktree from the unfinished-run refusal altogether, that departs from D7's list of refusals and needs an `Added after Discussion` entry recorded through the parent first.
      I checked this fix with `lyx loom validate-plan --overlay` (cards 13, 14 and 15 with their proposed target lists laid over the plan, baseline `{}` first): no new blocking finding.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/14-weft-integration-across-anchors.md:21
    summary: "integrateWeft moves the checked-out weft branch with reset --keep outside destroy.go, which PATTERN-fabric-git forbids and TestNoDestructiveBypass_FabricengineProductionSource rejects through its .ResetKeep( token"
    fix: >-
      In card 14, add `Create: plan:internal/fabricengine#resetWeftToIntegrated -> func resetWeftToIntegrated(rec *Mutations, weftPath, onto string) error`, and change the Intent sentence on line 21 so `integrateWeft` moves the branch to the whole result through it, instead of calling `reset --keep` itself.
      State that `resetWeftToIntegrated` is a `destroy.go` executor with a leading `rec *Mutations`, as card 15 states for `restoreMovePathsToHead`, running the shared pipeline first: containment of the weft worktree, ownership as a registered linked worktree, dirtiness as the tracked changes on the paths the move rewrites, no force, and only then `reset --keep`, as `DropLocal` does for its branches.
      Add to `TestIntegrateWeft` a row: a tracked change in the working tree on a path the result rewrites refuses at the dirtiness gate with the branch unchanged.
      I checked the target change with `lyx loom validate-plan --overlay` together with F3's and F1's target changes: no new blocking finding.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/13-move-record-stops-weft-writers.md:39
    summary: "no card names how the move record is removed, though cards 15 and 18 remove it; a bare os.Remove in a new fabricengine file fails the same destructive-bypass guard unless cmd/lyx/destructiveguard_test.go gains an allowlist entry"
    fix: >-
      In card 13, add `Create: plan:internal/fabricengine#RemoveMoveRecord -> func RemoveMoveRecord(weftPath string) error` and `Edit: cmd/lyx#destructiveGuardAllowlist`.
      In card 13's Intent, say `RemoveMoveRecord` is the one remover of the record, called by card 15's `Apply` failure path and `Finish` and by card 18's create, a tolerant `os.Remove` of `MoveRecordPath` in the new file `internal/fabricengine/moverecord.go`, since `internal/state` has no delete primitive, as `ClearPushFailure` removes the failed-push record.
      Say `destructiveGuardAllowlist` gains an entry for `internal/fabricengine/moverecord.go` whose reason is that the `os.Remove` deletes fabric's own move record in the weft's lock directory, fabric-internal metadata, never operator content.
      Then reword the three removal sites (card 15 lines 60 and 63, card 18 line 39) to call `RemoveMoveRecord`.
      I checked the target changes with `lyx loom validate-plan --overlay`: no new blocking finding.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/17-fabric-anchor-verb.md:52
    summary: "card 17 gives an end state only for Apply errors; an error between BeginAnchorChange and Apply, or a failed restore inside Apply, leaves the record standing with dobbies ended, and the prescribed restart is refused by card 16's gate"
    fix: >-
      Two edits.
      In card 15 (line 60), say `Apply` returns `landed` false only once the record is removed and the pair is wired at `A` again; when `restoreMovePathsToHead`, the `A/.lyx` rename or the rewire fails, `Apply` keeps the record, releases the weft write lock and returns the error naming the re-run `lyx fabric anchor <slug> --set <B>`, which recovers a not-landed record by resetting the move's paths to `HEAD`.
      In card 17 (lines 52 and 53), decide the restart by the record, not by `landed` alone: an error from the pass wait, the parent lock acquisition, the dobby end or the log sink close, which all run after `BeginAnchorChange` wrote the record and before `Apply`, and an `Apply` error that left the record, each restart no dobby (card 16's gate refuses a start while the record stands), release the parent lock, call `Close` and return the error naming the same re-run; only an `Apply` error with the record gone restarts the dobbies it ended.
      Add `TestChangeAnchor` rows with the fake: a parent-lock acquisition error after `BeginAnchorChange` and a dobby-end error each start no dobby and name the re-run; an `Apply` error with `landed` false and the fake's record still standing starts no dobby.
---

### [BLOCKING] The unfinished-run refusal loops on a worktree another machine owns

**Location:** `_lyx/plan/17-fabric-anchor-verb.md:44`, with `_lyx/plan/11-origin-reconcile-stops-on-marker-change.md:36-37`
**Issue:** Focus point 6 asks whether each refusal names a next step that progresses.
The owner refusal does: card 17 lets the `--set <upstream tip's marker>` through.
The unfinished-run refusal does not, for the same machine.
A run's `status.json` is durable under `_lyx` (`internal/shedrun/paths.go`, `StatusFile`), so a non-owner machine's checkout of the weft holds the owner's status as of its last pull, which is stale whenever the owner finished the run and re-anchored after this machine last pulled.
The weft pull is then stopped by the marker change (card 9), so the status can never refresh.
The chain from there:
- `lyx fabric anchor <slug> --set <upstream marker>` refuses on the stale unfinished status and names `lyx batten cancel <slug>`.
- `judgeOrigin` in `internal/battencli/cancel.go` returns `cancelOwnedElsewhere` for an owned-elsewhere slug, naming `lyx adopt <slug>` first.
- `lyx adopt <slug>` stops on the `anchor_stopped` weft (card 11 adds it to `stoppedBranches`), and the stop names the `--set`.
- The `--set` refuses again.
A cancel on the owner finds nothing to cancel, since the owner's run is finished.
Nothing in the plan breaks the cycle, and the decision record asks for each refusal to name its way forward (`PATTERN-refusal-way-forward`).
Round 1 raised this loop as its F2 and the fixer answered it differently (fixer report, F2): it kept `lyx batten cancel` as the step, on the evidence that `cancel.go:384-391` refuses an owned-elsewhere slug only when the worktree is absent or origin is unreachable.
That evidence does not hold.
Lines 384-391 are the first of two owner checks; the second is the `EntryOwnedElsewhere` case at `cancel.go:396-405`, which runs for a present worktree with a reachable origin and returns `cancelOwnedElsewhere`.
`classifyEntry` in `originreconcile.go` puts `owned_elsewhere` ahead of `stopped`, and the owner it compares comes from origin's fetched board, so an entry another machine owns is `owned_elsewhere` whatever its weft's state, and card 11's `anchor_stopped` change does not reach it.
Card 11's `judgeOrigin` change lets a cancel through "an entry stopped only by `anchor_stopped`", which an owned-elsewhere entry never is.
Card 17's own parenthetical, that the `--set` after the cancel goes to the upstream marker, assumes the cancel is accepted here.
No existing test in the plan reaches this combination: `TestChangeAnchor` has the owner rows and the unfinished-run rows separately.

### [BLOCKING] `integrateWeft` moves the weft branch with `reset --keep` outside `destroy.go`

**Location:** `_lyx/plan/14-weft-integration-across-anchors.md:21`
**Issue:** Focus point 5 asks whether any deletion or working-tree restore the plan prescribes goes through a `destroy.go` executor.
Card 15 does it right for `restoreMovePathsToHead`.
Card 14 does not: `integrateWeft` "moves the branch with `reset --keep` to it", which rewrites tracked files of the weft working tree and index, and the card names no executor.
`PATTERN-fabric-git` allows destructive primitives only in `destroy.go`, behind containment, ownership, dirtiness and force.
`cmd/lyx/destructiveguard_test.go` lists `.ResetKeep(` and `.CherryPick(` among its banned tokens, scanned in every `internal/fabricengine` file but the allowlisted ones, so an `integrateWeft` that calls `ResetKeep` fails that guard.
The card's `Uses` cites `cherryPickOnto`, the existing `destroy.go` executor, but its Intent describes a plumbing build followed by a bare branch move, not a call to it.
`DropLocal` and `restoreReplayTip` in `destroy.go` are the precedents for a gated `reset --keep`.

### [MEDIUM] No card says how the move record is removed

**Location:** `_lyx/plan/13-move-record-stops-weft-writers.md:39`, with `_lyx/plan/15-anchor-change-engine.md:60,63` and `_lyx/plan/18-beam-create-at-an-anchor.md:39`
**Issue:** Card 13 creates the record's type, path, reader and `MoveInProgress`, and no remover.
Cards 15 and 18 say "removes the record" in three places (`Apply`'s failure path, `Finish` and `Add`).
`internal/state` has no delete primitive (`deleteMergeState` in `mergestate.go` says so), so the removal is an `os.Remove`, a banned token of `TestNoDestructiveBypass_FabricengineProductionSource` outside `destroy.go` and the allowlisted files.
The existing record removals each sit behind a per-file allowlist entry with a stated reason (`failedpush.go`, `mergestate.go`), and the plan edits neither the allowlist nor `destroy.go` for the new one.
An implementer following the cards as written either fails the guard or invents the route; the plan should state it, as it does for the marker writer and the `.lyx-anchor` write containment in card 6.
This is MEDIUM rather than BLOCKING because the cards do not prescribe the violating call, only leave the mechanism open.

### [MEDIUM] End state after a failure past `BeginAnchorChange` is stated only for `Apply`

**Location:** `_lyx/plan/17-fabric-anchor-verb.md:52`, with `_lyx/plan/15-anchor-change-engine.md:60`
**Issue:** Focus point 2 asks whether each prescribed failure path names an end state and a landed re-run, and whether the restart can pass the gate.
For the `Apply` errors the plan is clear: `landed` false removes the record and restarts, `landed` true leaves the record, restarts nothing and names the re-run.
Two gaps remain.
First, `BeginAnchorChange` writes the record (card 15) before the pass wait, the parent-lock acquisition, the dobby end and the log-sink close (card 17 lines 48-51).
D7 says a failure before the commit removes the record and restarts the dobbies step 2 ended; card 17 gives that only for `Apply`.
An error from `parentlock` acquisition or `runproc.EndSparing` after some dobbies ended leaves the record standing, so card 16's `ensureOne` refuses every restart, and the verb's error does not name the re-run (only `MoveInProgress`'s text on a later `lyx beam start` does).
Second, card 15 line 60 returns `landed` false after a "restore" without saying what happens when the restore itself fails; if the record stays, card 17's restart on `landed` false is refused by the gate it set up two cards earlier.
Both are fixed by deciding the restart from the record's presence and naming the same re-run for every error that leaves it.

## Rejected

- Focus point 1, every weft pull or fast-forward path: `PullWeft` has two production callers, `Fabric.pullWeftArm` in `pull.go` and `Fabric.fastForwardWeft` in `parenttips.go`.
  Card 9 guards both with the marker comparison and card 13 adds the standing-record stop to both, so `FastForwardPair` in `landingshed/finalize.go` reaches no unguarded pull.
  The replay inside `settleUnmovedWeft` and `retryWeftPush` runs behind the same checks, and the origin reconcile's two weft moves (`ffWorktree`, `replayWeft`) are reached only through `originMover.move`, which card 11 guards.
  `Fabric.Merge`'s `syncSideBeforeMerge` fast-forwards only the warp side, so it is no weft path.
- Focus point 3, the drop-local reset: card 9's `reset` mode of `weftAnchorStop` ignores ancestry and stops whenever the two tips' anchors differ, and card 10 names the ahead-with-an-unpushed-re-anchor row in `TestDropLocal`.
  The `--set` it then names moves the weft back to origin's anchor and the adopt proceeds, so that case does not loop.
- Focus point 4, create at an anchor: card 18 names `moveWeftContent` as the one move step shared with card 15 and states that the record is removed after Add's own wiring, launchers and origin record and before `createSteps`' push, workspace and dobby steps; the order is stated.
- Focus point 6, invalid anchor targets: every `validateAnchorTarget` refusal in card 15 names a directory or checkout change and then the same verb, and none of those changes meets another refusal of the verb.
  The owner refusal's `lyx adopt <slug>` step is fine when the weft is not stopped; only the unfinished-run case in F1 loops.
- A time-of-check gap between card 9's fetch plus `weftAnchorStop` and `PullWeft`'s own `git pull --ff-only` fetch is real but needs a second push landing in a window of seconds; not raised.
