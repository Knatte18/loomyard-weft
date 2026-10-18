---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:54
    summary: D1 keys the per-slug teardown lock on the pause slug, so prime-side pushes that touch none of the pair's worktrees hold the pair's teardown lock and every batten teardown waits on, or stands Stuck behind, its own records push
    fix: In D1, split the two roles of the hidden slug. Keep the pause slug as now (the pair's slug for a push from the pair, the caller's gate slug for a push from the prime), and state that a child takes the pair's teardown lock only when the worktrees it pushes are that pair's own (a push from the pair); a push of the prime's records side, such as batten's seam and lyx batten cleanup, takes no teardown lock. Reword the sentence at line 58 to say which children take the lock, and add to the Teardown acceptance criterion that batten's status push spawned at the transition into Worktree-Teardown does not delay or refuse that row's Remove.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:62
    summary: D1 leaves the detached child's working directory inside the task worktree for loom's seam, so on Windows the child pins the tree Remove deletes and its cwd-anchored log sink can recreate .lyx/logs there, contrary to the Windows teardown criterion
    fix: In D1, add that fabric's spawn entry point starts every detached push child with its working directory outside both of the pair's worktrees (the hub directory), since the hidden slug flag now carries the pair identity the child's cwd carried, and that the child's logs go to a directory it is told outside the pair (such as fabricengine.HubLogsDir). Change line 62 to say that neither the lock file nor any child's working directory or log file sits in a tree Remove deletes.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:114
    summary: D4's trust bound for command keys lives in batten's setting lookup alone, while Command is a generic registry engine and the declared PATTERN-command-row makes it every recipe's side-action row; loom's Env.Setting resolves over the pair's full stack, tracked worktree layer included
    fix: In D4, move the bound into the Command producer's contract - the producer reads its command key through a lookup the recipe's owner tells it for executable keys, which resolves over the template and the board default layer only, and a recipe that tells none gets a refusal naming the way forward - and extend the declared PATTERN-command-row text with the same rule, e.g. Adding a side-action row to a recipe - a Command row over its config keys, its command read from the template and board default layer only, never a new engine. Add an acceptance criterion that a Command row whose owner tells no such lookup refuses before running anything.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:139
    summary: The PATTERN-fabric-destruction-chokepoint constraint places the local remove path's skipped origin steps in destroy.go, but that pattern confines only destructive primitives there, and Remove's step order and the archive push live in remove.go, weftwiring.go and archive.go
    fix: Reword the constraint to - PATTERN-fabric-destruction-chokepoint - the records-disposable dirtiness kind and deleteRemoteBranch's local-mark check live in internal/fabricengine/destroy.go; Remove's local-path step order and archiveWeftTip's mark check stay with their current callers.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:47
    summary: D1 names coalescePush as the blocking site to change, but that primitive also drives Bolt.Sync over the board's own push lock, and the record leaves open whether the board sync takes the try-lock and pending-flag protocol
    fix: In D1, state that the try-lock and pending-flag protocol applies to fabric's records push lock (fabric.push.lock under a records worktree's lock directory) only, and that Bolt.Sync's blocking coalescing over board.push.lock is unchanged; name the change site as the records-side push lock acquisition rather than coalescePush itself.
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:102
    summary: D3 states the local mark never applies to a pair created without --local, but the mark is a plain per-branch git config key any process in the repository can set, and one surviving mark makes a pair drop its records without archive or --force and silences its pushes
    fix: In D3's bound on remove, replace the sentence at line 102 with a bound that holds - the mark is written only by fabric add --local; a key set by hand on an ordinary pair's branch makes that pair local, so its remove archives nothing and drops its pending records, and its pushes publish nothing - and add that lyx fabric remove reports on its result that it took the local path and which branch carried the mark.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:135
    summary: D4's bound says no row's failure halts the run, while the same decision makes an unreadable setting an error whose way forward is lyx batten run, which stops the run
    fix: Change line 135 to - No command's outcome (a non-zero exit, a timeout, a Windows percent refusal) halts the run; an unreadable setting stops it at the row with the error above.
  - id: F8
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:54
    summary: The detached child's new hidden slug flag decides which pause entry it waits under and which teardown lock it takes, and D1 states no bound for a child given a wrong or empty slug
    fix: In D1, after the sentence introducing the hidden flag, add its bound - only fabric's spawn entry point sets it; a hand-run child given a wrong or empty slug waits under that slug's pause entry instead of the pair's and takes no teardown lock of the pair, so Remove does not wait for it; the worktree-existence check under the lock still keeps it from writing at a removed path.
---

### [MEDIUM] Prime-side pushes hold the pair's teardown lock

**Location:** `_lyx/discussion/decision-record.md:54`, and line 58.

**Issue:** D1 gives the child one hidden slug for two jobs.
The first job is the pause entry it waits under.
The second is the teardown lock it takes shared ("A child given a slug takes that lock shared for its whole run").
For a push from the prime, the slug is the caller's gate slug.
`battencli.wire` sets that slug to the pair's (`pauserecord.SetGateSlug(slug)` at `internal/battencli/wire.go`).
So batten's status push carries the pair's slug, and so does `lyx batten cleanup`'s, and both push only the prime's records worktree (`PushAnchored` over `RecordsWorktree(location)` in `internal/battencli/commitstatus.go`).
Remove deletes none of what those children touch, yet it waits for them.
Batten commits status, and now spawns this child, on every transition, including the one into Worktree-Teardown.
So every batten teardown first waits out a network push of the prime's records.
Under a pause on the pair, that push holds the lock past `StatusPushLockWait`, Remove refuses, and the row stands Stuck because of a push the teardown never needed to wait for.
That extra wait also cuts against the Goal's per-row-cost aim.

### [MEDIUM] The child's working directory stays inside the task worktree

**Location:** `_lyx/discussion/decision-record.md:62`; the Teardown acceptance criterion.

**Issue:** `fabricengine.SpawnDetachedPush` (`internal/fabricengine/spawn.go`) sets no `cmd.Dir`, and `proc.Detach` sets none on Linux or Windows, so the child inherits its spawner's working directory.
Loom's and darn's seams run in the task pair, so their child's cwd is inside the task worktree, one of the trees Remove deletes.
On Windows, a live process's cwd pins that directory, so Remove's worktree removal can fail while a child is still exiting: a child that released the teardown lock, or a late one that found the lock held and is exiting with its Warn.
Line 62 claims "no handle stays open in one on Windows", and that holds only for the lock file.
The logger's durable sink also resolves from the cwd (`resolveSinkDirLocked` in `internal/logger/sink.go`) and runs `os.MkdirAll(<anchor>/.lyx/logs)` on the first log line.
A late child's own exit Warn is a log line, written before or after the deletion, which works against "leaves nothing at the removed paths".
The hidden slug flag already replaces what the cwd told the child (the pause slug comes from `hubgeom.PauseGeometry` over the cwd today), so the cwd no longer has to be the pair.

### [MEDIUM] The command trust bound lives in batten's lookup, not in `Command`

**Location:** `_lyx/discussion/decision-record.md:114`, D4's Bound, and the declared `PATTERN-command-row`.

**Issue:** For batten, the D4 focus question has a clear answer, and it is not a defect.
A command key resolves through `configengine.Dirs` with only `Board` set.
`LayerFile` then applies the default layer alone (`internal/configengine/layers.go`), so no layer a task branch or adopted origin branch writes is read.
The Bound and the acceptance criterion "A command key in the task worktree's tracked `batten.yaml` is not run" both say so.
But D4 registers `Command` as a generic `shedrecipe` engine, and the declared `PATTERN-command-row` makes it the required shape for every recipe's side-action row.
The record leaves the layer choice to each recipe owner's `Env.Setting`.
Loom's `Env.Setting` is `configreg.SettingLookup(fabricengine.ConfigDirs(location))` (`internal/loomcli/wiring.go`).
`loom` is a per-worktree module, so that stack includes the pair's tracked worktree layer, which an agent in the pair can write through its `_lyx/config/` junction.
The first `Command` row outside batten would therefore run a command its own pair can set, and nothing in the producer or the PATTERN text stops it.
The bound is real for batten only, and the new edge applies to every recipe.

### [LOW] Destruction-chokepoint constraint misplaces orchestration

**Location:** `_lyx/discussion/decision-record.md:139`.

**Issue:** `pattern/PATTERN-fabric-destruction-chokepoint.md` confines destructive primitives and the `rec *Mutations` recorder to `destroy.go`.
`Topology.Remove`'s step order lives in `internal/fabricengine/remove.go`, `commitPendingRecords` and `archiveWeftTip` in `archive.go`, and `removeWeftWorktree` in `weftwiring.go`.
Skipping the archive push or the pending-records commit is a non-destructive decision.
A plan writer who follows the constraint literally would move Remove's orchestration into `destroy.go`.
What belongs there is the new records-disposable dirtiness kind (beside `dirtyScopeAll`) and `deleteRemoteBranch`'s mark check.

### [LOW] Unclear whether the try-lock change reaches the board's coalescing push

**Location:** `_lyx/discussion/decision-record.md:47`.

**Issue:** D1 says "Today `coalescePush` takes the push lock with the blocking `lock.AcquireWriteLock`" and bounds "the live pushers per records worktree".
`coalescePush` (`internal/fabricengine/coalesce.go`) is shared: `Bolt.Sync` drives it over `board.push.lock` (`internal/fabricengine/bolt.go`).
`_board` is a worktree of the records repository, so "per records worktree" reads as covering it.
The record never says whether the board sync gains the try-lock and pending flags.
`PushAnchored` and `PushPairAnchored` take the same records lock through `acquirePushLock` rather than `coalescePush`, so naming `coalescePush` also misses where the synchronous holders acquire it.

### [LOW] The local-mark bound states a premise, not a bound

**Location:** `_lyx/discussion/decision-record.md:102`, with line 90.

**Issue:** The mark downgrades two guards.
Remove archives nothing and discards pending records without `--force` through the records-disposable kind.
Every push skips the branch.
Line 90 makes one surviving mark enough.
Line 102 says "The mark never applies to a pair created without `--local`".
The mark is a per-branch git config key, though, and any process in the repository can write it, so nothing enforces that sentence.
The attack-surface rule asks the Decision to say what bounds the downgrade, and here the bound is only that no lyx code writes the key outside `--local`.
The result does not show that a remove took the local path, so a hand-set mark goes unnoticed.

### [LOW] "No row's failure halts the run" contradicts the setting error

**Location:** `_lyx/discussion/decision-record.md:135`, against line 122.

**Issue:** Line 122 makes an unreadable setting "an error naming the key and the way forward: correct it, then `lyx batten run <slug>`".
That way forward only makes sense if the run stopped.
Line 135's Bound then says "No row's failure halts the run."
The two cannot both hold as written.

### [LOW] The hidden slug flag has no stated bound

**Location:** `_lyx/discussion/decision-record.md:54`.

**Issue:** The new hidden flag picks both the pause entry the child waits under and the teardown lock it takes.
Under the rubric's attack-surface rule, a new flag states what it lets through.
A child run by hand with a wrong or empty slug skips the pair's pause and Remove's exclusion.
D1 states neither that only the spawn entry point sets the flag nor what such a child can still do.

## Focus verdicts

- D4 command keys: no layer an agent or an adopted origin branch writes reaches batten's command keys, and the Bound and an acceptance criterion say so; the gap beyond batten is F3.
- D1 teardown: the exclusion lives in a per-slug lock under `fabricengine.HubScratchDir`, outside both worktrees; a late child checks under that lock and exits; Remove's wait is bounded by `StatusPushLockWait` with a refusal and its way forward. Defects: F1 (who takes the lock) and F2 (the child's cwd and log sink on Windows).
- D3 local remove: no guard is left to refuse a used local pair; `--force` answers dirtiness only; the warp branch keeps `deleteWarpBranch`'s gate and `CodeBranchKeptReason`; a local remove commits no pending records, and the records-disposable kind lets `removeWeftWorktree` drop them; `RemoveRefusal`'s out-of-pathspec dirtiness refusal still applies; either surviving mark makes a pair local; cleanup's orphan-weft archive push checks the mark. The local weft branch's delete gate is `dirtyCheckedOutBranch`, which needs no archive, so it holds. Remaining: F6.
- D1 and D4 premises: each "today" claim matches the code. No no-op push skip exists in `statuscommit.New` beyond batten's marker. `coalescePush` and `acquirePushLock` block or wait. A child spawned from the prime resolves no pair slug from its cwd. Batten wires no `Env.Setting`. `--window` reaches Open-Window today through the hidden `--open-terminal` flag, which the record replaces with a told forced-row set rather than claiming one already exists.
