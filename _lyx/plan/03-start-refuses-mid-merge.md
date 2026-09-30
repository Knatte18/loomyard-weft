# Card 3 — start-refuses-mid-merge

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#newLoomCLI`
- `internal/loomcli#loomCLI.runDriverSpawnAndWait`
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#newTestSpawnAndWaitReceiver`
- `internal/loomcli#TestNewLoomCLI_SetsBothInjectedSeams`
- `docs/overview.md`

**Create:**
- `internal/loomcli/start_midmerge_test.go`

**Uses:**
- `plan:internal/shedrun#StartMergeInProgressKind`
- `plan:internal/fabricengine#MidMerge`
- `plan:internal/fabricengine#MidMergeState`
- `plan:internal/fabricengine#MidMergeKind`
- `plan:internal/fabricengine#MidMergeNone`
- `plan:internal/fabricengine#MidMergeParked`
- `plan:internal/fabricengine#MidMergeForeign`
- `internal/shedrun#StartNotParkedKind`
- `internal/loomcli#loomCLI.resumeParkedDriver`
- `internal/loomcli#loomCLI.runHaltedAtHandBack`
- `internal/loomcli#fakeDriverPaneProbeFull`
- `internal/loomcli#parkedStrands`
- `internal/loomcli#newResumeBranchReceiver`
- `internal/loomcli#acquireTestBootstrapLock`
- `internal/loomcli#assertBootstrapLockReleased`
- `internal/loomcli#noopLockHeld`
- `internal/loomcli#markerExists`
- `internal/loomcli#writeTestRunState`
- `//internal/fabriccli/envelope.go`

**Intent:**
Make `lyx loom start` (and so its root alias `lyx start`) refuse on its envelope, before putting a driver to work, when the pair carries an unfinished merge.

Seam.
Add a field to `loomCLI` holding the mid-merge probe, typed `func(*lyxcwd.Location) (fabricengine.MidMergeState, error)`, with a doc comment saying why it is a seam (the Test Tier Purity Invariant bars an untagged test from building a real mid-merge pair).
`newLoomCLI` sets it to `fabricengine.MidMerge`, next to `spawnWatchdog`, so both constructors carry it; extend `TestNewLoomCLI_SetsBothInjectedSeams` to assert it is non-nil, and rename it `TestNewLoomCLI_SetsInjectedSeams` with its doc comment updated to match, since "Both" is wrong once it pins a third field.
`newTestSpawnAndWaitReceiver` sets it to a fake answering `MidMergeNone` with no error, so every existing `runDriverSpawnAndWait` test keeps its current behaviour.

Refusal, in `runDriverSpawnAndWait`.
After `mustSpawn` is computed, read the park marker's presence once (the stat the resume branch already does) and derive whether this invocation would put a driver to work: `mustSpawn`, or a live driver strand with the marker present (the resume branch).
Only then call the probe with `c.location`, before the stale-marker removal, the resume line, or any spawn.
A live strand with no marker is never probed, so a live working driver is attached to exactly as today and a live unparked driver over a hand-back keeps its `driver_not_parked` refusal.
The decision reads only `mustSpawn`, strand liveness and the marker, never the seed's driver value (Driver Choice Single-Site Invariant).
Every refusal releases `bootstrapLock` before reporting and returns false, exactly like the surrounding failure returns, and leaves the park marker on disk.

- Probe error: `output.Err(out, err.Error())`, no kind — an undetermined answer spawns nothing.
- `MidMergeParked`: `output.ErrFields` with `kind` = `shedrun.StartMergeInProgressKind` and `conflicts` = the probe's `Conflicts` (an empty JSON list, never null, when none remain).
  The message names the remedy consistently with `internal/fabriccli/envelope.go`'s conflict text: a fabric merge is in progress in this worktree; resolve each listed path, mark it resolved with `"lyx fabric merge-stage <path>..."`, then run `"lyx fabric merge --continue"` (or `"lyx fabric merge --abort"` to discard the merge), then re-run `"lyx loom start"`.
- `MidMergeForeign`: the same kind and `conflicts` field, with a message saying a git merge, cherry-pick or squash that fabric did not start is in progress in this worktree; conclude or abort it with git, then re-run `"lyx loom start"`.
  It names no fabric merge verb, since those refuse foreign state.

Neither message, nor any identifier or comment in `internal/loomcli`, names the two sides of the pair (Fabric Vocabulary Invariant).
Update `runDriverSpawnAndWait`'s doc comment to state where the check runs and which branches it guards.

Help text.
In `startCmd`'s `Long`, step 3 gains the refusal: before spawning or resuming a driver, start refuses with the kind "merge_in_progress" when the worktree carries an unfinished merge, naming the conflicted paths and the remedy (the fabric verbs for a fabric merge, git for one fabric did not start); a live driver that is working is left alone.
Keep the `Long` text's existing plain-text wrapping style.

Docs.
In `docs/overview.md`'s loom module entry, next to the `driver_not_parked` sentence, add one line for the non-retryable `merge_in_progress` refusal: when `start` would spawn or resume a driver over a pair with an unfinished merge it refuses, listing the conflicted paths under `conflicts` and naming the remedy.
Semantic line breaks.

Bundled test, `internal/loomcli/start_midmerge_test.go` (untagged; fakes only, no git), driving `runDriverSpawnAndWait` through the existing receivers with a fake probe that records whether it was called and returns a canned state or error.
Each refusal row decodes the envelope and asserts `ok` false, the exact `kind`, the exact `conflicts` list, the message substrings named below, that the starter and sender were never called, that the go arm's driver log (`loomengine.LoomDriverLog`) was not created, and that `assertBootstrapLockReleased` holds.

- Spawn, llm arm, no strand, fabric-parked with conflicts → refuses with `merge_in_progress`, `conflicts` equal to the canned paths, message naming `lyx fabric merge-stage`, `lyx fabric merge --continue` and `lyx fabric merge --abort`.
- Spawn, go arm, no strand and a free run lock → same refusal, no detached spawn.
- Fabric-parked with no conflicts left → same kind, `conflicts` an empty list (assert the raw JSON carries `[]`).
- Foreign state → same kind, message naming git and not `merge-stage`.
- Spawn with a stale park marker on disk → refused, and the marker is still on disk afterwards.
- Parked live driver (live strand plus marker, via `newResumeBranchReceiver`) over a mid-merge pair → refused, no resume line sent, marker still on disk.
- Live driver, no marker, run `running` → the probe is never called, `start` succeeds exactly as today.
- Live driver, no marker, run halted at a hand-back → the probe is never called and the refusal kind stays `driver_not_parked`.
- Probe error → refused on the envelope with no `merge_in_progress` kind, nothing spawned.
- The real `fabricengine.MidMerge` wired as the probe over the receiver's own temp-dir location, which is no pair → refused as a probe error, nothing spawned.
- Clean pair (`MidMergeNone`) on the spawn path → proceeds past the check to the starter exactly as today.

**ImpactSummary:** runDriverSpawnAndWait has 1 production caller (start's bootstrap in start.go); the new loomCLI field is set by newLoomCLI, used by both Command() and StartAliasCommand(), and left nil by every other test literal, none of which reaches runDriverSpawnAndWait except through newTestSpawnAndWaitReceiver (given a MidMergeNone fake); startCmd's change is help text only; batten's childNotParked, the one out-of-package reader of the start kind, treats the new kind as a hard error.

**Commit:** 3: loom start: refuse to spawn or resume a driver over an unfinished merge
