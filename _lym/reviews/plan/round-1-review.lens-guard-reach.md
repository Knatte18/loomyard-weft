```yaml
lens: guard-reach
outcome: reported
```

Task: work the guard-reach lens over cards 1–10, checking whether each guard, refusal, branch and test row the plan adds can be reached.

**Per-guard list (reachable or not)**
- Card 2: the `SkipOrch` branch returns before `startOrch`. Reachable through `startHarness.runWith`.
- Card 3:
  - `Read` on an absent record answers false: reachable.
  - An undecodable record wraps `ErrDecode`: reachable (TestRead fixture).
  - `ReadState` maps `running` to `interrupted` on a dead pid or other ticks: reachable.
  - `ResumeStep` folds `owner` or earlier into `reconcile`: reachable.
- Card 5:
  - `paused_by_adopt` from an `adopt` entry, or from a legacy `adopt-hold` entry: reachable.
  - The `ownerHaltResume` clause under an adopt pause, and on a plain `halted` line: reachable. A run this machine already owned can halt on its parent's owner and is never a pause target.
- Card 6:
  - These stops are reachable through the world: `pauseErr` stops at `pause`, a failed deploy stops at `deploy`, a failed `SetOwners` stops at `owner`, and `stepDone` ends `stopped` on a held-back project.
  - A refused pair add joins no `HeldBack`: reachable.
  - The `NoOrch` pass-through: reachable.
  - `pauseEntries` rows (none, owner conversion, other caller, listed skip, beam conversion, beam with none): reachable.
  - The replace-in-place `StepRecord` and the outcome seeding on resume: reachable through card 7 rows.
- Card 6, the standing-move refusal (`refuseStandingMove`): reachable on a fresh run, but **unguarded on a resume past `owner`** (finding 3).
- Card 7:
  - `validateRequest` refuses before the lock: reachable.
  - A held `adopt.lock`: reachable.
  - Other subjects refused: reachable.
  - Fresh on a done record or no record, resume on a dead pid or a stopped record: reachable.
  - The worker-side `follow` verdict, declared an unreachable wiring guard: **reachable from on-disk state** (finding 1).
  - The worker's read of an undecodable record: reachable, **no named test** (finding 4).
- Card 8:
  - `--discard` with a subject refused: reachable.
  - `discard` refused while another holder has the lock: reachable.
  - `discard` removing an undecodable record: reachable.
  - `spawnWorker` refused under a test binary: reachable.
  - `runsInOrch`: reachable.
  - Follow of a later-started record with the same subjects, and refusal of one with other subjects: reachable (two resumers race for the lock).
  - A record started before `since` is not followed: reachable (stale done record, worker refused).
  - The "no record, read once more" branch: reachable (`--discard`).
  - Interrupted, stopped and done ends, and the cap: reachable.
  - On the follow-verdict path (nil `exited`), the "record names another pid" state is **reachable and unguarded** (finding 2).
  - `runAdopt` on an undecodable record: **no named test** (finding 4).
- Card 9: the per-state rows of `adoptSectionOf` are reachable, and so is the undecodable read joining the verb's failures (integration test).

**Findings**

1. MEDIUM, design, card 7 `judgeStanding` / `runWorker` step 3 (and card 3 `WorkerAlive`).
   - Card 7 calls the worker-side `follow` verdict a fail-loud wiring guard that no state can reach. It is reachable from on-disk state.
   - `WorkerAlive` is planned as `proc.IsAlive` plus ticks, with "a record without ticks reads alive on the pid alone".
   - On Linux, `IsAlive` answers true for a zombie (signal 0 succeeds; `IsRunning` is the zombie-aware read). The zombie's stat ticks still match the record. So a new worker that takes the lock just after the old worker exits, before it is reaped, reads a live worker and gets the `follow` error.
   - On Windows, `StartTime` always reports false, so records carry no ticks. A pid reused after a reboot reads alive. A `running` record then reads `running` for good:
     - the worker refuses;
     - a plain-shell caller follows the unrelated process uncapped;
     - D9's `running` way forward does not name `--discard`.
   - The wiring-guard exemption fails because refusal-spec requires a state "unreachable from any on-disk state a run can produce".
   - Fix: `WorkerAlive` uses `proc.IsRunning`, and `proc.CreationTime` where no ticks exist. Alternatively, treat the verdict as a transient refusal naming `lym adopt --discard`, with a reaching row.
   - Evidence: `internal/proc/proc_linux.go:19-24,30-34`; `internal/proc/proc_windows.go:47-50,59-61`; `contracts/specs/refusal-spec.md:36`; card 3 line 198; card 7 lines 552-554.

2. MEDIUM, design, card 8 `runAdopt` follow branch and `follow`.
   - On the follow verdict, `follow` gets a nil `exited`, which never closes.
   - Its "the record names another pid" branch only fires once `exited` is closed. So the follower ignores any record of another pid.
   - Path: the followed worker ends `done`, then another adopt takes the lock and rewrites the record within one wake (up to 8s). The follower never sees its worker's end state again.
   - In a plain shell (`followCap` zero) it loops forever. In an agent session it ends at the cap with a misleading `still running`.
   - Fix: on the nil-`exited` path, a record of another pid ends the follow, either as the followed worker's loss or with "the adopt you followed ended; `lym beam status`". Add a reaching row in `TestFollow`.
   - Evidence: card 8 lines 662, 682-686, 689.

3. LOW, design, card 6 `stepReconcile` / `runSteps` and card 7 resume.
   - Today `adopt` refuses a standing anchor change before the reconcile (`refuseStandingMove` over slugs and local runs).
   - Card 6 never names where that refusal goes. Its only hint is "today's reconcile and its refusals" in `stepReconcile`.
   - Card 7's `validateRequest` list omits it. So a resume past `owner` reaches `stepDeploy` with no standing-move check.
   - D1 names that exact interruption source: `lym fabric anchor` ending the worker mid-step.
   - `BeamUp` refuses a beam with a standing move itself, so only the deploy and the check start are unguarded on that path.
   - Fix: run the standing-move check in `runSteps` before the first step of every run, resumes included, with a `TestRunWorker` row.
   - Evidence: `internal/adoptcli/adopt.go:236-244`; `internal/orchcli/start.go:486-489` (`beamUpRefusal` with `moveWayForward`); card 6 line 421; card 7 lines 543, 561.

4. LOW, consistency, card 7 `runWorker` step 3 and card 8 `runAdopt` step 2.
   - Both read the record before judging it.
   - An undecodable `adopt.json` (`ErrDecode`) is a refusal branch on both verbs, and its way forward is `lym adopt --discard`.
   - No `TestRunWorker` or `TestRunAdopt` row names that input. Only `TestRead` (card 3) and the `beam status` integration test reach the error, and neither reaches it through these verbs.
   - Fix: one row each ("an undecodable record refuses before any spawn or step, naming `lym adopt --discard`").
   - Evidence: card 7 line 550; card 8 line 661; card 3 lines 202-203.

**Tool calls**
- Read the `IsAlive` / `StartTime` definitions in `internal/proc/proc_windows.go` and `proc_linux.go`. The manifest had only their locations, and finding 1 needed their zombie and no-ticks behaviour.
