---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:D4 (the push sentences) and Constraints PATTERN-owner-machine bullet
    summary: "D4 gates a task pair's push on the loom run's push verdict but names no home for that verdict and none of its exceptions or side effects, which today live unexported in internal/loomcli/prepush.go"
    fix: "In D4, after the sentence that says the verb spawns the push only when the reconcile's verdict lets a loom run's push go, add the decisions a plan writer cannot take from the record. (1) The verdict logic (`prePushCheck.guard`, `skipsCheck` and `stopReason` in `internal/loomcli/prepush.go`) moves to one shared place that both loom and `lym gate testmap` call, because `internal/gatecli` cannot reuse an unexported loomcli type; name that place (a function in `internal/fabricengine` taking the pair's location and slug is the fit, since the reconcile is fabric's). (2) The verb skips the check and pushes as the loom run does when pushes or git are off (`fabricengine.EnvSyncOptions`), when a pause covers the run, and when the pair is the prime; for a run standing at Finalize the verb takes the same skip, since Finalize judges origin itself. (3) The push also goes when the reconcile fails, origin is unreachable, or the read of whether the parent holds the run's work fails, so the push's own failure is recorded. (4) State what a hold does: the verb pauses no run and posts no creel notice, and prints one line naming the reason and the way forward (loom's stop pauses the run and notices the orch; a hand call must not). Keep the existing bound that a held verb pushes nothing and its commit waits for the pair's next push. Then make the Constraints PATTERN-owner-machine bullet say the same in one clause."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:D5 (the sentences from 'Fabric's weft pull first commits' through 'Bound') and the D5 acceptance criteria
    summary: "D5 claims ErrReplayDirty never stops on maps, but its own pre-pull commit puts those map paths into the replay's touched set, so a live pass rewriting one of them before the replay's dirtiness check still stops the pull; no bound or criterion names it"
    fix: "In D5, replace 'So neither `weftDirtyStop` nor the replay's `ErrReplayDirty` stops on them' with the exact claim. The commit leaves the weft one local commit ahead, so a weft that origin moved takes the replay path instead of the fast-forward, and the replay's dirtiness check (`replayRequest` in `internal/fabricengine/replay.go`, `dirtyTrackedOn(touched)`) covers exactly the paths of that commit. Add a Bound sentence: a map a live pass rewrites between D5's commit and that check stops the pull with `ErrReplayDirty`, as a `*WeftStopError` whose way forward is `lym fabric commit` then re-running the pull; the stop touches no state, and the next pull commits the rewritten map and goes through. Keep the existing 'replaced atomically on disk' sentence for the commit race. Add an acceptance criterion under D5 for the live-pass case: a map rewritten after the pre-pull commit and before the replay yields the stop with that way forward and loses no map write; and one for the replay path: a prime sync whose origin weft moved and whose only uncommitted change is a map replays the maps commit onto origin's tip with no dirty stop."
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:Acceptance criteria, D4 entries, against Constraints PATTERN-pair-remote
    summary: "The Constraints say a task pair's testmap push carries the code branch and the prime's stays records-only, but no D4 criterion pins either, only the held case is covered"
    fix: "Add two D4 acceptance criteria next to the existing ones: a testmap call in a task pair that changed maps spawns the detached records push with the code branch along; a testmap call in the prime that changed maps spawns a records-only push. Phrase them as observable outcomes (what the spawned push carries), not as call sequences."
---

### [MEDIUM] D4's push gate names a verdict it does not place, bound or finish

**Location:** `_lym/discussion/decision-record.md` D4 ("In a task pair it first runs the run-scope origin reconcile ...") and the Constraints `PATTERN-owner-machine` bullet.

**Issue:** The declared PATTERN change says `lym gate testmap`'s push in a task pair reconciles run-scope origin first.
D4 states that, names `fabricengine.ReconcileOrigin`, and states the bound for a hold (nothing pushed, the commit waits).
What it leaves open is the verdict itself.
The only code that turns a run-scope report into "the push goes" or "the push is held" is `prePushCheck` in `internal/loomcli/prepush.go`, and it is unexported there.
Read in full, it carries three things D4 does not say the verb takes or drops:
- Its skip conditions: pushes or git off (`EnvSyncOptions`), the pair is the prime, a pause covers the run, the run stands at Finalize (`skipsCheck`).
  A hand call in a task has no loom run or status file, so which of these apply to it is a real choice.
- Its pass-through cases: a failed reconcile, unreachable origin, a failed read of whether the parent holds the work.
  `PATTERN-owner-machine` lists them for the loom push; D4 only says "lets a loom run's push go".
- Its stop side effects: a stop pauses the run (`pause` or `pauseOwned`) and posts a notice to the nearest orch.
  If the verb borrows the verdict, a plan writer must decide whether a hold also pauses and notices.
  A hand testmap call pausing a run is a new routing edge, and D4's bound ("pushes nothing") does not say it does not.
Rubric lens: the attack-surface question, a routing edge introduced with part of its bound unstated; and a fact the plan writer must go find (where the verdict lives and what it does), which the record can settle now.
The fix settles the placement and the exceptions and says a hold has no side effect beyond its own output line.

### [MEDIUM] D5 overclaims that the replay never stops on committed map paths

**Location:** `_lym/discussion/decision-record.md` D5 and its criteria.

**Issue:** D5 commits every uncommitted `_lym/testcover/` path before the weft moves, "So neither `weftDirtyStop` nor the replay's `ErrReplayDirty` stops on them".
Checked in `internal/fabricengine/pull.go` and `replay.go`:
- After that commit the weft is ahead of origin by one commit, so a weft origin moved no longer fast-forwards; `settleUnmovedWeft` takes the `len(ahead) != 0` branch into `replayWeft`.
- `replayWeft` builds `touched` from the paths of the commits ahead, and `replayRequest` makes the dirtiness probe `dirtyTrackedOn(touched)`.
  So the map paths D5 just committed are exactly the paths whose later dirtiness stops the replay with `ErrReplayDirty`.
- A live pass writes a map per round and a package more than once, so a rewrite of a just-committed map between D5's commit and that probe is possible, and the pull then stops.
The atomic-replace sentence covers the commit racing a write; it does not cover this window.
The stop is recoverable (the way forward is `lym fabric commit`, then re-run), which is the bound the record should state, with a criterion.
The listed D5 criteria cover only the case with no live pass.
Without the bound, a plan writer reads D5 as "a pull never stops on a map", and a test written to that claim would be flaky against a running pass.

### [LOW] D4's push has constraints but no criterion for what it carries

**Location:** `_lym/discussion/decision-record.md` Acceptance criteria D4 entries; Constraints `PATTERN-pair-remote`.

**Issue:** The Constraints bullet states the task pair's push carries the code branch and the prime's is records-only.
The D4 criteria cover the commit, the killed call and the held push, but nothing observes the pushes that do go, so the constraint is held by review alone while a test could pin it.

## Rejected

- D4 and D1 lock and path sources: not a defect.
  The writer works through the weft path and the readers through the junction, but `lock.TrackedFileLockPath` resolves the `_lym` link with `EvalSymlinks` and takes the lock under the resolved checkout's `.lym/locks/`, so both reach one lock file.
  The Constraints `PATTERN-lym-files` bullet and D4 say the same thing (write and build locks via `lock.TrackedFileLockPath`, only the exit record never-tracked under `.lym/testcover/`).
- D9: the record makes every task call register under the task's slug (`runproc.EndSparing` ends the slug's records), so the teardown reaches a named-package rebuild; the prime's `--upkeep` registration under `HubOwner` and the dobby's `TestMapRunning` read of `HubOwner` records keep the two passes from blocking each other.
  Not a defect.
- D7 readers: the record states what the redundancy step's map age and drift read (`BuiltAt` and `Files` of the last write) and keeps them; `PlanUpkeep` moves `Files` to the tree but does not set `Changed` for a files-only difference, so an unwritten current map keeps older file hashes under its older commit, consistently.
  The job guard in `internal/gatecli/testmappackage.go` (`planned.Commit != head` on the file) is the one D7 retires for such a job.
  A drift line for a file edit that touched no test is the documented meaning, not a staleness the plan ruled out.
  Not a defect.
- D11: the record names the lock (the package's build lock, taken without waiting) and the bound (re-check for a map, skip a held package).
  The commit it names, `5e41ee2b`, is the commit all the on-disk bolt maps record (checked across every `map.json` under `<bolt>/.lym/testcover/`), and `git diff --name-only 5e41ee2b 79f0eb4a` lists only two files under `docs/research`, so "no Go change" holds.
  Not a defect.
- Open risks: none of the four lines is answered by an Added-after-Discussion entry (those settle the extra branch and the raddle split, and both lines are gone).
  Not a defect.
