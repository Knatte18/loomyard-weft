```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. Trace the code paths that already run against each decision's state, and judge whether each decision's result survives them.

**Paths traced, per decision**

- D13 vscode cut: in-verb workspace regeneration (`regenerateAfterVerb`, every exit of `Add` including rollback at `add.go:480`, the deferred `regenerateHubWorkspaceNow` in `Remove` at `remove.go:269`, `anchorchange.go:539`). **Breaks.**
- D13 vscode cut: external `RegenerateHubWorkspace` callers (`ideengine/spawn.go:135`, `beamcli/dobbydeps.go:429`, `loomcli/pawlwiring.go:108`, `adoptcli/deps.go:110`). Survives if they pass content.
- D13 vscode cut: the existing global hooks `InstallWorkspaceContent` and `InstallPairFolderNamer` (`hubworkspace.go:91`, `:115`). The record does not mention them.
- D13 shedengine/shedrun cut: `codetipsettle.go:89-90` writes a `shedengine.Status` into shed's status file under `shedrun.StatusLock` and commits it. **Breaks.**
- D13 shedengine/shedrun cut: `takeoverwrite.go:146-260` builds a `shedengine.Takeover` entry, writes it into the run's status file under shed's lock, commits and pushes it with retries and `rewriteOverOrigin`. **Breaks.**
- D13 shedengine/shedrun cut: the public readers `OriginRunStatus`, `OriginParentRunStatus`, `OriginBattenStatus` and `ReadUpstreamTakeover` (`originreads.go:32`, `handover.go:183,205`, `takeover.go:117`) return shed's types. Their git reads must stay in fabric, and the record does not place them. **Breaks.**
- D13 shedrun cut: the path-only uses (`RunsRootRel`, `TreeRunFileRels`, `RunDirRel`, `IsReserved`, `PrimeRunLock`) at `addanchor.go`, `add.go:710`, `pull.go:587`, `forkrecords.go`, `replay.go`, `originreconcileended.go`, `originreconcilemoves.go:44`. Survives through D13's path and lock seam.
- D2/D3 teardown child: `RunTeardownChild` reads a `remoteTeardownJob` from stdin, and the parent wrote that job (`remoteteardown.go:61-86`, `childHold.finish` at `:394`). **Breaks or is undecided.**
- D2/D3 teardown child: the pending-teardown record is written from the same job (`writePendingTeardown`, `keepFailedTeardown`, `SettleGonePairTeardown`) and read by `PendingTeardowns` for `fabriccli` status and prune and for `battencli` cancel. Survives only if the job shape is kept, which the record does not settle.
- D2/D6 teardown child, test path: under a test binary, `holdTeardown` runs the job in process with the parent's location (`remoteteardown.go:340`). The seam tests never reach the child's own snapshot. **Breaks the test/production match.**
- D2 pair push: `DetachedPush` is reached from `fabriccli/weft_verbs.go:279` and takes the `PushSpawn` paths and slugs. It re-reads the worktrees and the pause and teardown locks itself. Survives.
- D2 background pusher: `runpusher` spawns it (`runpusher/start.go:161`), not fabric. It reads `RunCommitsAhead`, `RefusedPushStands` and `IsPushOwed`. Survives, but the record misnames who spawns it.
- D4 Mutations: `fabriccli/envelope.go:30,56,83` prints `rec.Entries()` as the `mutations` JSON array, and `fabriccli/remove.go:227` iterates `report.Mutations.Entries()`. **Breaks** if the executor records action kinds that are unrecorded today.
- D4 live-pair probe: `OriginSeams.LivePair` at `originreconcilemoves.go:527`, with adapters in `pairteardown/originseams.go:135` used by every `ReconcileOrigin` caller. Survives.
- D3 teardown seam: `OriginSeams.Teardown` goes through `pairteardown` back into `Topology.Remove`, a whole fabric verb with its own locks. It survives as a seam-call boundary if the re-read follows it.
- D7: the `lymcwd` enforcement maps and `boardkit`/`envkit` retargets. Survives, since the record names them.
- D12: `hubforge` imports `fabriccli`, `fabricengine` and `hubreconcile`. Survives under D12's `hubforge` exemption.

**Findings**

1. **BLOCKING · design · `_lym/discussion/decision-record.md` D13 (vscode bullet).**
   D13 says the IDE module "passes [the workspace] as bytes". But fabric regenerates the workspace inside its own verbs, after their state changes: every `Add` exit including rollback, `Remove`'s deferred regeneration after teardown, and the anchor change. No caller holds those bytes before the verb runs, so "bytes" cannot keep D5's behaviour. The existing seams are installed hooks, `InstallWorkspaceContent` and `InstallPairFolderNamer`, and the record does not mention them. The fix is a builder seam that fabric calls with its post-verb folder list, bounded like D13's other seams.
   Evidence: `internal/fabricengine/hubworkspace.go:78,115,163,172`, `internal/fabricengine/remove.go:269,507`, `internal/fabricengine/add.go:480`, `internal/fabricengine/anchorchange.go:539`.

2. **BLOCKING · design · `_lym/discussion/decision-record.md` D13 (shedengine/shedrun bullet).**
   D13 treats shed's data as facts that reach fabric through a read seam: "run status and takeover", "run paths and run locks". Fabric also writes shed's records today:
   - `codetipsettle` rewinds a run's status file to the previous transition's `shedengine.Status` and commits it.
   - `takeoverwrite` writes a `shedengine.Takeover` entry into each branch's status file, commits it, pushes it with retries, and rewrites it over origin on rejection.

   The public readers `OriginRunStatus`, `OriginParentRunStatus`, `OriginBattenStatus` and `ReadUpstreamTakeover` return shed's types from git reads that only fabric may run. D13's seam vocabulary of flags, refs, paths, lock handles and verdicts cannot carry the shape of a record fabric must write, or of one it reads out of a commit. The record does not decide whether shed supplies a codec, fabric moves opaque bytes, or these verbs move out. Without that decision, the D13 guard and D5 cannot both hold.
   Evidence: `internal/fabricengine/codetipsettle.go:89-131`, `internal/fabricengine/takeoverwrite.go:146,168,217,253-260`, `internal/fabricengine/originreads.go:32-79`, `internal/fabricengine/handover.go:183,205`, `internal/fabricengine/takeover.go:117`.

3. **MEDIUM · design · `_lym/discussion/decision-record.md` D2 (detached child paragraph).**
   D2 says the parent hands the child only "the target, such as the slug and the refs it must carry, never an action list", and the child decides from its own snapshot. The real job carries more than a target:
   - parent decisions: `DeleteWeftOnOrigin` (leased on the origin tip the archive read), `DeleteTaskOnOrigin` and the `Rerun` way forward;
   - facts the child cannot re-read: `WeftTip` and `TaskTip`, whose local branches the parent has already deleted.

   The pending-teardown record persists this job, and `fabric status`, `fabric prune` and `batten cancel` read it. The record also does not say that under a test binary `holdTeardown` runs the job in process on the parent's location. That means D6's seam tests never exercise the child's own snapshot. The record should state which job fields stay parent-decided, that the pending record keeps its shape, and that the in-process path follows the same per-child snapshot rule.
   Evidence: `internal/fabricengine/remoteteardown.go:61-86,340-345,394-410,482-520`, `internal/fabricengine/pendingteardown.go:64,73,142`, `internal/fabriccli/weft_verbs.go:169`, `internal/fabriccli/fabric.go:855`, `internal/battencli/cancelcmd.go:294`.

4. **MEDIUM · design · `_lym/discussion/decision-record.md` D4 (Mutations sentence) against D5.**
   D4 says "the executor records each action it ran in `fabricengine.Mutations`". `fabriccli` prints `rec.Entries()` as each verb's `mutations` JSON array, and `lym fabric remove` iterates the report's entries. If the executor records action kinds that are unrecorded today, such as a push, a seam call or a ref create, every verb's printed output changes, which D5 forbids. The record should say that each action kind maps to today's mutation kinds, or that it records none where none is recorded today.
   Evidence: `internal/fabriccli/envelope.go:30,56,83`, `internal/fabriccli/remove.go:227`, `internal/fabricengine/mutation.go:128-216`.

5. **LOW · consistency · `_lym/discussion/decision-record.md` D2 (list of detached children).**
   D2 names "the background pusher's push" as a detached child that fabric spawns. `internal/runpusher`, a module above fabric, spawns it and reads fabric through `RunCommitsAhead`, `RefusedPushStands` and `IsPushOwed`. Fabric's own detached children are the teardown child and `SpawnDetachedPush`.
   Evidence: `internal/runpusher/start.go:87-176`, `internal/runpusher/run.go:64-77`, `internal/fabricengine/spawn.go:64`.

**Tool calls**

1. `remoteteardown.go` `remoteTeardownJob` and the pending-teardown funcs and callers: the manifest lacked the job's fields and the pending record's readers.
2. The `hubworkspace.go` funcs and the `RegenerateHubWorkspace` callers: the manifest lacked fabric's in-verb regeneration sites.
3. `codetipsettle.go:55-110` and `takeoverwrite.go:130-268`: the manifest showed shed symbol uses, not that they are writes.
4. The external readers of `Mutations` and `runpusher`'s fabric calls: the manifest lacked the output readers and who spawns the pusher.
