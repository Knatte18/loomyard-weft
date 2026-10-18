---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 Bound, and the sentence giving the command keys the trust class of deploy
    summary: batten is a per-worktree module read over the task worktree's stack, so a command key with CategoryExecutable is still settable in the task branch's tracked batten.yaml, which agents and an adopted origin branch can write; deploy and darn verify are HubWide and settable only at the board default layer, so the stated trust class and bound are false
    fix: In D4, give the command keys a home no task branch can write, for example a HubWide module or a rule that the Command producer reads command keys from the board default layer only, while the switch keys keep reading the task worktree's stack. Rewrite the Bound to name exactly which layers may set a command and why a task branch cannot, and add an Acceptance criterion that a command key in the task worktree's tracked batten.yaml is not run.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D1 Bound, the sentence on fabricengine.Remove holding the push lock through the remote steps
    summary: the push lock file is <records worktree>/.weft/fabric.push.lock, inside the worktree Remove deletes, so holding it through teardown keeps an open handle in the tree being deleted (Windows refuses the delete) and on Linux a child arriving after the unlink creates a fresh lock file and never contends with Remove
    fix: In D1, decide where Remove's exclusion against children lives, so that it outlives the records worktree, for example a per-slug lock under the hub-level .lyx that both the detached child and Remove take, or Remove taking the existing lock only until no child is in flight and releasing it before removeWeftWorktree. State the release point relative to the worktree teardown, and name both platforms in the matching Acceptance criterion.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3 Bound on remove
    summary: the weft is never merged (fabricengine doc.go:52 and 1027), so any status or record commit on a local pair's weft is contained in no other branch for life; every used local pair then refuses without --force, and --force also answers dirtiness, so the guard turns into a routine force that drops records and uncommitted work
    fix: In D3, redefine the guard over what can be lost, for example containment of the warp branch only, with the weft's records judged separately by an explicit rule (records of a local pair are disposable, or kept by a local archive the guard names). Give the guard its own flag instead of reusing --force, so answering it never also answers dirtiness. Update the Acceptance criterion so a local pair that ran a shed run removes as the new rule says.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D1 Bound, the clause that a child started after Remove fails on the missing worktree and warns
    summary: CoalescePushBothAt calls ensureWeftLockDirAt, whose os.MkdirAll recreates <removed records worktree>/.weft before anything checks the repo, so a late child leaves a directory at the removed sibling's path that a re-add of the same slug refuses on
    fix: In D1, state that the detached child checks its worktrees exist before it creates the lock directory and exits with a Warn when they are gone, and add to the teardown Acceptance criterion that a child spawned after Remove leaves nothing at the removed paths and a re-add of the slug succeeds.
  - id: F5
    severity: MEDIUM
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1 Bound, the sentence on live children bounded by the absorbing lock
    summary: coalescePush takes the blocking lock.AcquireWriteLock, so a child that finds the lock held queues instead of leaving its commits to the holder; the record asserts the false premise and then leaves the behaviour to the plan, and a try-lock built naively loses the commit of the run's final transition
    fix: In D1, state the current behaviour as fact (children block and queue), decide the bound this task builds, and state its race rule, for example a child that fails a try-lock writes a pending flag under the lock directory that the holder re-checks after its last no-progress iteration before releasing, so no committed transition is left unpushed. Drop the clause that leaves this to the plan.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1 Bound, Remove waits for the pair's push lock before its first mutation
    summary: the wait has no stated bound, and a child stuck in git holds the lock as long as git runs, so Remove and batten's Worktree-Teardown row can stall indefinitely
    fix: In D1, give Remove's wait a bound (for example StatusPushLockWait) and state the refusal it returns when the wait runs out, with its way forward (retry the remove once the push finishes, or the step that ends a stuck child), and add that refusal to the Acceptance criteria.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1 Bound, one child per committed transition and never per no-op transition
    summary: statuscommit.New calls Push after every nil Commit, and both seams discard CommitAnchoredPaths' committed flag, so a clean-path no-op still pushes today; loom's and darn's seams have no marker skip, so the record states as existing a skip the code does not have
    fix: In D1, add that the seams' Commit dependency returns whether a commit landed and that the statuscommit core spawns the push only when one did, and keep the Acceptance criterion that a no-op transition spawns nothing as the test of that change.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1 Bound, the detached child honours a hub pause waiting under the run's pause entry
    summary: batten sets its gate slug in-process with pauserecord.SetGateSlug, but the child is a fresh lyx process whose gate is built from its cwd, the prime, so its slug is empty; a pair-scoped pause no longer holds batten's or cleanup's status push, while loom's seam, running in the pair, keeps it
    fix: In D1, state how the child learns the pair's slug, for example a hidden slug flag the spawn entry point passes and the child's gate install honours, or state that batten's status pushes are held by hub pauses only and drop the claim that they wait under the run's entry.
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3, --force then answers both dirtiness and unmerged commits
    summary: deleteWarpBranch runs with force false, and Remove's contract says force never answers the landed-work gate, so --force passes the new refusal while the warp branch with unlanded commits is still kept and only the weft is deleted; the record does not say which rule wins
    fix: In D3, state what happens to each branch of a local pair under the new guard and under --force, and either keep the existing warp keep-with-reason (scoping the new guard to the weft) or state that the local remove path deletes the warp branch past that gate and why.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D3, the unmerged-commit guard against Remove's pending-records commit
    summary: Remove commits the sibling's pending records before its archive step, and that commit is itself uncontained; with the guard among the first refusals the fresh pair passes and its pending records are then committed and deleted with no archive, and with the guard after the commit it contradicts every existing refusal still runs first
    fix: In D3, state where the guard runs relative to commitPendingRecords and how pending records count toward it, for example the guard counts uncommitted record changes as unmerged work, so a local pair with pending records refuses without the guard's answer.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D3, the skip sits where every fabric push passes
    summary: Topology.Cleanup archives every orphan weft branch by pushing a tag through archiveWeftTip, a raw gitexec push outside the gitrepo push primitives, and D3 exempts only lyx fabric remove, so an orphaned local weft branch is published by lyx fabric cleanup --apply
    fix: In D3, state that the archive tag push and remote branch deletes also honour the local mark, naming fabric cleanup beside remove, and add an Acceptance criterion that cleanup of a local pair's orphaned weft branch pushes nothing.
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4, settings read through Env.Setting when the row is reached
    summary: batten wires no Env.Setting today (only loomcli does, over its own location); batten is wired in the prime before the task worktree exists, and run_settings land in the task worktree's local layer, so a lookup wired like loom's would silently stop run_settings from switching rows
    fix: In D4, state that batten wires Env.Setting over the task worktree's config stack, resolved lazily when the row is reached as switchedActions' readFlags does today, and reconcile that with F1's home for the command keys.
  - id: F13
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria, the under-100-ms row against D1 and D2
    summary: the criterion counts everything beyond the row's process for a switch-off row, which includes D1's synchronous status commit, while D2 licenses cutting only costs beyond the commit, so the criterion may be unreachable with no Decision allowed to reach it
    fix: Either reword the criterion to measure beyond the status commit and the row's process, or extend D2 so the commit's cost is in scope of the cut when it alone breaks the budget.
  - id: F14
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4, --window forces Open-Window's switch and an empty command under --window warns
    summary: today the override sits in batten's Enabled closure via c.openTerminalFlag; a generic producer reading the switch through Env.Setting has no channel for the override or for knowing --window is in force to choose Warn
    fix: In D4, state where the override enters, for example batten wraps its Setting lookup so open_window reads true under --open-terminal and tells the producer a forced switch, and state that the Warn keys on that told fact.
  - id: F15
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 Placeholders, each value quoted for cmd on Windows
    summary: no quoting stops cmd.exe expanding %NAME% even inside double quotes, so a value containing percent signs is not passed literally and the quoting bound does not hold for cmd
    fix: In D4, state the cmd rule for percent signs (escape each percent as the cmd quoting requires, or refuse a value holding one with a way forward), and add a percent sign to the placeholder Acceptance criterion.
  - id: F16
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3, the mark lives and dies with the branch and a marked pair's remove runs no origin step
    summary: a pair whose weft or warp branch was deleted by hand loses that branch's mark, and the record does not say whether a half-marked pair counts as local, so remove may run the archive and origin probes of an ordinary pair on it
    fix: In D3, state the rule for a half-marked pair, for example a pair counts as local when either surviving branch carries the mark, and that a pair with no surviving mark is removed as an ordinary pair.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D3 report it skipped, and the Acceptance criterion that a commit in a local pair reports the local branch skipped
    summary: the detached child is never waited and has nil stdio, so a skip found inside it can only be logged, never reported to Fabric.Commit's or a seam's caller, which records only that a push was spawned
    fix: In D3, state that the spawn entry point checks the mark first and spawns no child for a local pair, recording the skip on the caller's result, or reword the criterion so the detached paths report the skip in the log.
  - id: F18
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Open risks, the row-entry-gate and pair-from-parent bullets
    summary: the row-entry-gate bullet is future deliberation about another note's design, not a risk to this task, and the pair-from-parent bullet restates the Out item; neither carries a requirement the plan writer needs
    fix: Move the row-entry-gate bullet to support-log.md's Interview, and delete the pair-from-parent bullet from Open risks, since Out already names that note.
---

The record is well shaped and its premises are mostly grounded, but three of its bounds fail against the code it rests on.
The command trust class in D4 is weaker than the record claims.
D1's teardown exclusion lives inside the directory it tears down.
D3's remove guard fires on every used local pair, so `--force` becomes routine.
Several D1 premises about current behaviour are stated as fact and are not.

### [BLOCKING] Command keys settable from the task branch

**Location:** `_lyx/discussion/decision-record.md` D4 Bound

**Issue:** `deploy` (`internal/landingshed/config.go:101`) and darn's `verify` (`internal/darnengine/config.go:139`) are `HubWide`, and `LayerFile` in `internal/configengine/layers.go` gives a HubWide module no worktree layer under a board, so they are set at the board default layer alone.
Batten's spec has no `HubWide`, and its rows read `ConfigDirs(taskLocation)` (`internal/battencli/wire.go` `switchedActions`).
A `CategoryExecutable` command key is then still read from the task worktree's tracked `batten.yaml`, which the task's agents and an origin branch Add adopts can write.
A later `lyx batten run` or `goto` reaching the row runs that command with the operator's rights, and the stated bound covers only `run_settings`.

### [BLOCKING] Remove holds a lock that lives inside the tree it deletes

**Location:** `_lyx/discussion/decision-record.md` D1 Bound

**Issue:** `RecordsLockDirPath` is `filepath.Join(recordsWorktreePath, weftLockDirName)` (`internal/fabricengine/recordsgit.go:53`), and the lock file is `fabric.push.lock` there (`recordsgit.go:34`).
Holding it through `removeSibling` → `removeWeftWorktree` (`internal/fabricengine/remove.go:233,323`) keeps an open handle in the directory being deleted, which Windows refuses.
On Linux the unlinked inode stops excluding anything: a child arriving after the unlink makes a fresh lock file.

### [BLOCKING] The local remove guard makes --force routine

**Location:** `_lyx/discussion/decision-record.md` D3 Bound on remove

**Issue:** `internal/fabricengine/doc.go:52` and `:1027` state that the weft is never a merge participant and `main-weft` never advances on landing.
Every status commit on a local pair's weft is therefore contained in no other branch forever, so the guard refuses every pair that ran anything, which is exactly the batch pairs D3 is for.
`--force` then answers both this guard and dirtiness, so the routine removal also discards uncommitted work, with no archive.

### [MEDIUM] A late detached child recreates the removed records worktree's directory

**Location:** `_lyx/discussion/decision-record.md` D1 Bound

**Issue:** `CoalescePushBothAt` calls `ensureWeftLockDirAt` (`internal/fabricengine/recordsgit.go:59-62`), whose `os.MkdirAll` runs before any repo check, so a child spawned after teardown leaves `<removed records worktree>/.weft` behind.
Add refuses an existing records worktree path, so the record's "fails on the missing worktree and warns" is false and a re-add of the slug breaks.

### [MEDIUM] The absorbing-lock bound rests on a false premise and is left to the plan

**Location:** `_lyx/discussion/decision-record.md` D1 Bound

**Issue:** `coalescePush` takes `lock.AcquireWriteLock` (`internal/fabricengine/coalesce.go:24`), which blocks, so every child queues behind a hung holder.
The record asserts the opposite and then hands the fix to the plan, leaving a decision open.
A try-lock without a re-check loses the commit of a run's final transition, which has no next transition to carry it.

### [MEDIUM] Remove's wait for the push lock is unbounded

**Location:** `_lyx/discussion/decision-record.md` D1 Bound

**Issue:** The record names no bound for the wait, and a holder lasts as long as git does (`internal/fabricengine/pushanchored.go`, `PushAnchored` doc), so a stuck child stalls Remove and batten's Worktree-Teardown row.

### [MEDIUM] The no-op transition skip does not exist today

**Location:** `_lyx/discussion/decision-record.md` D1 Bound

**Issue:** `internal/statuscommit/statuscommit.go` calls `deps.Push()` after any nil `Commit`, and both seams discard the `committed` flag (`internal/battencli/commitstatus.go:85-88`, `internal/loomcli/wiring.go:190-193`).
The record states the skip as existing, so the plan gets no Decision to thread `committed` through.

### [MEDIUM] Batten's detached child loses the pair's pause entry

**Location:** `_lyx/discussion/decision-record.md` D1 Bound

**Issue:** `internal/battencli/wire.go:742` sets the gate slug in batten's process only.
`SpawnDetachedPush` sets no `cmd.Dir`, so the child inherits the prime's cwd, and `InstallNetworkGate` (`internal/tendercli/gate.go`) builds an empty slug from it, through `internal/hubgeom/pausegeom.go:14`.

### [MEDIUM] --force semantics contradict the warp branch's existing gate

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:** `deleteWarpBranch` passes `force: false` (`internal/fabricengine/remove.go:447`), and Remove's contract at `remove.go:130-131` says force never answers that check, so the record's `--force` answers a refusal while the warp branch is still kept.

### [MEDIUM] The guard's order against the pending-records commit is unstated

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:** Remove commits pending records before the archive (`internal/fabricengine/remove.go:169-180`).
Run first, the guard passes a fresh pair whose pending records are then committed and deleted with no archive.
Run after the commit, it breaks D3's own "every existing refusal still runs first".

### [MEDIUM] fabric cleanup still publishes a local pair's orphan weft

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:** `internal/fabricengine/cleanup.go:225` calls `archiveWeftTip` on every orphan weft branch, a raw `gitexec` tag push outside the gitrepo push primitives, and D3 exempts only remove.

### [MEDIUM] Batten wires no Env.Setting, and the stack it must read is unstated

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** `Env.Setting` is nil-legal (`internal/shedrecipe/recipe.go:148-150`), and only `internal/loomcli/wiring.go:881` sets it, over its own location.
Batten is wired in the prime before the task worktree exists, while `run_settings` land in the task worktree's local layer (`internal/battenshed/seamchild.go:139-149`).

### [MEDIUM] The under-100-ms criterion counts a cost D2 may not cut

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria

**Issue:** For a switch-off row the criterion counts everything beyond the row's process, which includes D1's synchronous commit through `CommitAnchoredPaths`, while D2 cuts only costs beyond the commit.

### [LOW] The --window override has no mechanism under a generic producer

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** The override reads `c.openTerminalFlag` inside batten's `Enabled` closure (`internal/battencli/wire.go` `switchedActions`; `internal/battencli/cli.go:115,300`), which a generic `Command` producer does not have.

### [LOW] cmd expands percent signs inside quotes

**Location:** `_lyx/discussion/decision-record.md` D4 Placeholders

**Issue:** `internal/verifyrun/run.go:43-45` runs `cmd /C` on Windows, and `cmd` expands `%NAME%` inside double quotes, so a path holding `%…%` does not reach the command literally.

### [LOW] A half-marked pair's remove is undefined

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:** The mark dies with each branch, so a pair with one branch deleted by hand carries a mark on one side only.
`refuseWhenNothingRemains` (`internal/fabricengine/remove.go:383-399`) and the archive step then decide by rules the record does not give.

### [LOW] A skip inside the detached child cannot be reported to the caller

**Location:** `_lyx/discussion/decision-record.md` D3 and Acceptance criteria

**Issue:** `SpawnDetachedPush` (`internal/fabricengine/spawn.go`) never waits for the child and leaves its stdio nil, and `internal/fabriccli/fabric.go:518-519` records only that a push was spawned.

### [NIT] Two Open risks carry no risk to this task

**Location:** `_lyx/discussion/decision-record.md` Open risks

**Issue:** The `row-entry-gate` bullet speculates about another note's later design, and the `pair-from-parent` bullet repeats the Out item; neither carries a requirement, so the first moves to the support log and the second goes.

## Rejected

- Board-entry edits: no Decision, Scope item or Acceptance criterion asks the run to change this task's own board entry; the focus directive's check found none.
- D2's open-ended cut: the cut is bounded to batten's per-transition path and needs no further bound.
- The destruction-chokepoint Constraint misplacing the local remove path: `PATTERN-fabric-destruction-chokepoint` binds destructive primitives and gate checks to `destroy.go`, and the guard can live there as a dirtiness kind while `Remove` stays in `remove.go`, so the Constraint is satisfiable as written.
