```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims, bounds and ordering promises with concrete paths and interleavings.

**Claims attacked**

- D4: the export comes after the `env` module's exports, so it wins. **Held.** reed puts the PaneEnv exports ahead of the launch command (`internal/reedengine/spawn.go:125`).
- D6: the resume check runs before `Prepare`. **Held** (`internal/shuttleengine/run.go:589` comes before `:650`).
- D6: the orch's resume reaches the same check through the runner. **Held.** `resumeOrch` calls `StartSession`, which calls `start`, which calls `checkResume` (`internal/orchcli/start.go:806-808`, `run.go:541-589`).
- D6: the check records a copy it made beside the run directory's launched-account record. **Broken** (F-A).
- D6: a copied resume whose first turn ends on an API error other than a usage limit is a refused resume. **Broken** (F-C).
- D6: the copy is made before the live-holder test. **Broken, minor** (F-H).
- D7, step 1 and the PATTERN change: a relaunching record reads as live until the new GUID is written. **Broken** (F-B).
- D7: today's stop verb would record stopped. **Held** (`internal/shuttleengine/stop.go:48`, `:93-95`).
- D7: today `retryAPIError` re-sends a usage limit. **Held** (`internal/shuttleengine/apierrorretry.go:46`).
- D7, beams pin: "its pinned account now has a `blocked_until`" when the switch resolves. **Broken, ordering unstated** (F-F).
- D3: "two picks at once for one worktree end with one entry and one increment", against "replaces any entry". **Broken** (F-D).
- D5 bound: forged figures only push the pick to step 3. **Broken** (F-E).
- D2 prune with D9: the prime keeps an entry that a set or a switch writes. **Unproven** (F-G).
- D4: a reed relaunch keeps the account it launched under. **Partly broken** for the no-account case (F-I).
- D1: refusal of a tracked value also when the machine layer holds the key. **Held.** Each layer is read on its own.
- D5: `OpenLaunch` writes settings with no creel flag directory. **Held, but the case is vacuous.** A board directory is empty only outside a hub, where there are no accounts.
- D7 `account set` bound: any process that can run lym from a worktree's directory. **Held** as stated.

**Findings**

- **F-A**: BLOCKING, design, `_lym/discussion/decision-record.md:170`.
  - **Summary:** D6's "the check records a copy it made beside the run directory's launched-account record" has no run directory to write to on two of its three entry points.
    - The shuttle resume at start runs `checkResume` before `createRunDir` and `Prepare`.
    - The orch resume runs the same path.
    - `CheckResume(sessionID, workdir)` is given no run directory.
    - So the fresh-start fallback, and the orch watcher's "refused resume", cannot learn a copy happened. Only D7's in-run relaunch has a run directory.
  - **Evidence:** `internal/shuttleengine/run.go:589` (`checkResume`) comes before `:645` (`createRunDir`) and `:650` (`Prepare`); `internal/shuttleengine/engine.go:357` (signature); `internal/orchcli/start.go:806-808`.
  - **Fix:** say how the copy fact reaches the run. For example, the check returns it and the runner writes it into the new run directory after `createRunDir`; or `Prepare` records it from a told flag.
- **F-B**: BLOCKING, design, `_lym/discussion/decision-record.md:202` and the PATTERN changes bullet at `:290`.
  - **Summary:** the relaunching mark has no failure, restart or concurrent-stop path, and the PATTERN text makes attach and Wait read it as live with no strand and no bound.
    - **Step 1 fails after its record write.** `Stop`'s own contract is that a failed removal leaves the record over a live strand, and step 1 sits outside "a failure in steps 2 to 4". The record stays relaunching over the old live process forever.
    - **The Wait process dies between steps 1 and 4.** Hibernate or a crash is enough. The record reads as live while no strand is up, and nothing finishes the relaunch. An attach then waits on a strand reed does not track. After `2×StartupTimeoutS` any sibling `Start` sweeps the run directory, because the sweep keys on reed's strand table, not the record.
    - **A concurrent `lym shuttle stop <old guid>` during the window.** `stopRecorded` writes stopped across processes. Step 4 then saves the in-memory state with the new GUID and running, so the operator's stop is lost and the new strand runs.
  - **Evidence:** `internal/shuttleengine/stop.go:39-40`, `:87-101`; `internal/shuttleengine/run.go:1088-1110`; `internal/shuttleengine/rundir.go:355-363`.
  - **Fix:** state the step-1 failure outcome. State who resumes or settles an interrupted relaunch on attach, with a bound on how long relaunching reads live. State how a stop during the window wins over step 4.
- **F-C**: MEDIUM, design, `_lym/discussion/decision-record.md:171-174`.
  - **Summary:** the fallback fires on any non-usage-limit API error after a copy, such as an overload or a 500. D6's own Facts say a foreign organisation's thinking is dropped silently, so no API error is the expected refusal.
    - The rule therefore discards a good resumed session and starts fresh on a transient error.
    - `retryAPIError` is told neither to count nor to re-send that turn end, so the transient error is never retried.
  - **Evidence:** `decision-record.md:176-178`; `internal/shuttleengine/apierrorretry.go:61-69`.
- **F-D**: MEDIUM, consistency, `_lym/discussion/decision-record.md:81` against acceptance `:318`.
  - **Summary:** D3 says the pick "replaces any entry the file holds for that name", so the second of two serialised picks for one worktree re-picks and increments again. The acceptance criterion demands one increment.
  - **Fix:** say what tells a duplicate pick from a recreated worktree.
- **F-E**: MEDIUM, design, `_lym/discussion/decision-record.md:151-152`.
  - **Summary:** the forged-input bound "bounded to the pick falling back to D3's step 3" understates the reach. D7's switch uses D3's step 2 eligibility.
    - Forged high figures on every other account make each limited worktree switch nothing and wait the limit out.
    - Forged low figures on an exhausted account make switches target it, so each relaunch lands on a limit again.
  - **Evidence:** `decision-record.md:191-192`.
- **F-F**: LOW, design, `_lym/discussion/decision-record.md:194-195`.
  - **Summary:** the beams-pin switch depends on `blocked_until` already being written, under D5's separate usage-file lock, before D4 resolves the relaunch. No order is stated.
    - If it is written later, or the write fails, the entry and the count increment are written but the worktree still resolves to the pin. The relaunch condition is then false, and the limit is handled as today with the counter skewed.
- **F-G**: LOW, design, `_lym/discussion/decision-record.md:71-72` with D9 at `:257`.
  - **Summary:** the record does not say the told reader of current worktree names includes the prime. A reader of task and beam worktrees would let every pick drop the prime's entry from a set or a limit switch. A beams-pinned prime would then resolve back to its blocked pin.
  - **Evidence:** no such reader exists today. I found only `dotgit.ListWorktrees` and `lymcwd.NamedWorktree`.
- **F-H**: LOW, design, `_lym/discussion/decision-record.md:165-168`.
  - **Summary:** the copy runs before the live-holder test. A resume later refused for a live holder has already overwritten the resolved directory's transcript and `<session>/` directory. That can happen to a file a live process appends to.
- **F-I**: LOW, design, `_lym/discussion/decision-record.md:117-118`, `:124`.
  - **Summary:** with no account, the stored command neither sets nor unsets the variable. reed reads PaneEnv afresh at each launch, so a relaunch after an `env` change runs under a directory other than the recorded effective one. D8's live readers then read the wrong directory.
  - **Evidence:** `internal/reedengine/spawn.go:73`, `:125`.

**Tool calls**

1. `run.go:1060-1140` and the attach grep: the orphan sweep and attach's outcome reads, which the manifest lacked, to test the relaunching window.
2. `rundir.go` `sweepOrphans`: its removal rule (directory age and reed strand table), which the manifest lacked.
3. A grep for a hub worktree-names reader and `lymcwd.NamedWorktree`, to check whether the prime is listed, which the manifest did not cover.
