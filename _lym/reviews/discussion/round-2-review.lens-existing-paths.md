```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing path that writes or reads the state D1 to D10 introduce or change.

**Paths traced, per decision**

- **D1, records first:**
  - `PushPairAnchored` (pushanchored.go:151-161): survives once its order is flipped.
  - `CoalescePushRequest` and `coalescePushSides` (coalesce.go:139-164), the detached push child that `teardownlock.go:111` runs, still push the code side first: **breaks**.
  - Callers that stay detached and send a pair through that child:
    - the anchor change's `SpawnStatusPush(pair)` (anchorchange.go:557): **breaks**;
    - `lym fabric sync` and `lym fabric add` through `spawnPush` and `spawnPairPush` (fabriccli/spawn.go): **breaks**;
    - fabric's commit seam, a fire-and-forget spawn (commit.go:152): **breaks** wherever D1 does not retire it;
    - `pairteardown/followup.go:107` and `tendercli/deps.go:148`: prime only, records only, survive.
  - `pushDeferral` in the remaining detached child marks `pushed[code]` before the records side (coalesce.go:157), so the code side lands first: **breaks**.
  - `respawnPairPush` (teardownlock.go:158): retired by D1, survives.
- **D1, pending flags left on disk:**
  - `runHeld` takes the owed sides' flags, so a flagged code side is carried by the next `PushPairAnchored` holder: survives.
  - The remaining detached child's `coalesceFlagged` also takes a code flag and pushes it code first: **breaks** (same root as above).
- **D1, renamed `PushOwed`:** `Act` and `Judge` (act.go:85, 203-204) only rename the field: survives.
- **D3, machine id in `machine.json`:**
  - `machinecli/set.go` `nextFile` rebuilds the file as `machinename.File{Name, Names}` (set.go:138), dropping any other key: **breaks**.
  - `machinecli/current.go` reads through `ReadFile`: survives.
  - `FilePath` returning `ok=false` (no user config dir) leaves no file to hold an id: **breaks** for that machine.
- **D3, comparisons of a recorded machine name:** the `boardengine.OwnedElsewhere` callers across beamcli, battencli, loomcli, orchcli, adoptcli, fronttask, pairteardown and fabriccli all move with "takes the registry": survives.
- **D5, `bolt-hold` slug entries, read by `pauserecord` coverage readers:**
  - `pairteardown.ReadRun`: `Held` is set from Bolt and the act rule checks `Held` before `PauseCovers`, so it survives.
  - `pauserecord.CoveredError`, rendered by loom's pause hold (pausehold.go:57), loom's driver resume (driverresume.go:109), shuttle's open (open.go:77) and shuttle's paused send (pausedsend.go:94): **breaks**.
  - `CoverageOf` sets `Held` only for `AdoptCaller` and `HoldCaller` (record.go:241-245), so a `bolt-hold` entry reads as a plain pause: **breaks** for every display or notice reader, such as battencli's dobby notice.
  - Hibernate's reconcile judges by caller identity (verbs.go:81): survives.
  - `tenderengine` resume (resume.go:147, 279): survives once D5's refusal is added.
  - The `pairteardown/skip.go:149` driver-check skip on a covered pair: survives, since D7 excludes covered pairs from the dobby's teardown.
- **D6, adopt record readers:**
  - `beamcli/adoptsection.go` and `beamcli/cli.go`: named in D6, survives.
  - `orchcli/watch.go`: named in D5, survives.
  - The fronttask `adopt-check` row runs `lym adopt check &lt;subject&gt;` (fronttask.go:34). D6 retires the worker and steps but names no `adopt check` verb: **breaks**, see finding 4.
- **D8, printed `parent_beam`:** the readers are boardcli, boardengine's store, the board skill and orch-role. All are named in D8, so it survives.

**Findings**

1. **BLOCKING, design. D1, the detached pushes that stay.** The surviving detached push child still sends a pair's code side before its records side, so D1's "the order holds for every caller" and the declared `PATTERN-pair-remote` text ("the code branch goes along after the records side") do not hold.
   - D1 reorders only `PushPairAnchored`. The child behind `SpawnStatusPush` and `CoalescePushRequest` (`coalesce.go:152-164`, reached from `teardownlock.go:111`) pushes code first, and its pause deferral leaves only the records flag (`coalesce.go:157`).
   - D1 keeps that child for the anchor change (`anchorchange.go:557`, a pair push), for `lym fabric sync` and `add` (`fabriccli/spawn.go`), and for fabric's commit spawn (`fabricengine/commit.go:152`).
   - Each sends every unpushed code commit of the pair, an agent's in-flight work included. After a takeover it does so without D2's takeover read.
   - This breaks the D1 criterion "a code side … never reaches origin before its records side, nor after a takeover entry naming another machine".
   - Fix: D1 states the detached child's order and its takeover read, or routes these pair pushes through the synchronous path.

2. **BLOCKING, design. D5, refusal wording of a `bolt-hold` entry.** `pauserecord.CoveredError` (`record.go:303-310`) words every slug entry's refusal as "run `lym tender resume &lt;slug&gt;`".
   - D5 makes that verb refuse for a slug only a `bolt-hold` entry covers, and the refusal still names that verb.
   - Callers that render it: loom's pause hold (`loomcli/pausehold.go:57`), loom's driver resume (`loomcli/driverresume.go:109`), shuttle's open (`shuttleengine/open.go:77`) and shuttle's paused send (`shuttleengine/pausedsend.go:94`).
   - D5 rewords only the act rule's held refusal, so these refusals name a way forward that refuses. That breaks `PATTERN-refusal-way-forward`.
   - Fix: D5 states that a `bolt-hold` entry's covered refusal names the hold's machine and D5's way forward.

3. **MEDIUM, design. D3, the machine id across a rename.** `lym machine name` rewrites `machine.json` from a freshly built `machinename.File{Name, Names}` (`internal/machinecli/set.go:138`), so it drops any key that struct lacks.
   - D3 puts the machine id in that file and has the same verb record the rename in the registry, but never says the rewrite keeps the id.
   - If the id is lost, D3's own bound applies: the machine gets a new id, and its old entry refuses its old and current names. A rename then turns the machine into a stranger to its own registry entry.
   - Machines that resolve no user config dir (`machinename.FilePath` returns `ok=false`, `machinename.go:42-51`) have no file to hold an id at all, and D3 is silent on them.
   - Fix: D3 states the id survives every rewrite of the file, and says what a machine with no machine file registers as.

4. **MEDIUM, design. D6, `lym adopt check` and the fix task's re-check.** D6 says "the task's landing runs the check again". That re-check runs through fronttask's `adopt-check` row, which spawns `lym adopt check &lt;subject&gt;` (`internal/fronttask/fronttask.go:21-34`).
   - D6 retires the worker, the record and the step machinery around today's check (adoptrecord importers: `adoptcli/steps.go`, `steprun.go`, `follow.go`, `worker.go`). Its kept-flags list names `--archives`, `--no-orch`, `--all` and `--here`, and never says `lym adopt check` stays.
   - Nor does D6 say that `check` reads the `adopt-fix` hold for the try count, clears the hold on green and starts `orchcli.BeamUp`.
   - So the one path that clears an `adopt-fix` hold is unstated.
   - Fix: D6 names `lym adopt check` as kept, with what it reads and writes under the Bolt hold.

5. **MEDIUM, design. D5, coverage readers misread a `bolt-hold` entry.** `pauserecord.Record.CoverageOf` sets `Held` only for `AdoptCaller` and `HoldCaller` (`record.go:241-245`).
   - Every coverage reader therefore reads a `bolt-hold` entry as a plain slug pause: the dobby's notice (`battencli/dobbynotice.go:61`), the dobby reads (`battencli/dobbyreads.go:172`), the batten overview (`battencli/overview.go:242`) and the tender report (`tenderengine/report.go:189`).
   - These display and notify it as an operator pause, with no hold machine named.
   - D5 says the legacy callers stay declared only for the upgrade's read, and does not move `Held` to `BoltHoldCaller`.
   - Fix: D5 states that `Coverage.Held` reads `bolt-hold`.

**Tool calls**

1. `grep` for adoptrecord importers and fronttask's adopt row: who reads the adopt record D6 retires, and what drives the fix task's re-check.
2. `grep` for `CoveredError`, `CoverageOf`, `ReadCoverage` and `Covering` callers: the readers of the slug entries D5 adds.
3. `grep` for `CoalescePushBothAt`, `CoalescePushRequest`, `SpawnStatusPush` and `spawnDetachedPushFn` callers, plus the anchor change's push: the detached pair pushes D1 keeps.
4. `grep` in machinecli for the machine file's writer, and in hibernateengine and tenderengine for entry handling: who rewrites `machine.json` (D3) and how hibernate and resume treat slug entries (D5).
5. `grep` for `parent_beam` readers: the readers of the printed value D8 changes.
6. `sed` on `teardownlock.go:95-111`, `commit.go:140-155`, `fabriccli/spawn.go` and `pauserecord/record.go:295-330`: the detached child's entry, the commit spawn, fabric's spawn sites, and `CoveredError`'s wording.
