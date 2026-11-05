```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I looked for what the diff forces outside its own lines. I found no defects.

**Per change, the set found and whether it is covered**

1. **New `run-ended` file at the run-directory root.**
   - Scans of the root: `ReadAgentActivity`, `ReadNewestRunEnd`, `sweepOrphans` and `waitmark`, `attach`, `carryover`, `sessionread`, `agentwait` and `stepstop` all call `os.ReadDir(root)`. Each of these files has an `IsDir()` check, so the stamp is skipped, as `engines-run.json` already is. Covered. I confirmed the check exists in each file by count only (one `IsDir()` per file), not that it guards the same loop as the `ReadDir`.
   - `sweepOrphans` (`rundir.go:349-352`) skips non-directories, so the stamp is never swept. Covered.
   - `enginesrun.go:45-46` also reads the root. It is the ledger's own reader, and I did not read how it treats other root files. Not covered by my reading.
   - `battencli/cleanup.go:224` scans the shed records and scratch roots, not the run-directory root. Unrelated.
   - The one other writer, `finalize`, is covered: `wait.go:1927` stamps from `filepath.Dir(run.runDir)`, the same root `runDirRoot` resolves.
2. **`ReadNewestRunEnd` result now includes the stamp.**
   - Its only production caller is `readNewestAgentEnd` (`wire.go:717`), reached through `DobbyRunNotices.NewestAgentEnd` (`dobbynotice.go:50`). The seam signature is unchanged, so `runnotice.go:270` and the `dobbyengine` fakes still hold.
   - Docs: the `ReadNewestRunEnd` doc comment and `shuttleengine/doc.go:488-489` are updated. The `NewestAgentEnd` seam comment in `runnotice.go:106` and `dobbyengine/doc.go:139` ("the end of its newest ended agent run") stay true.
   - Tests: `TestReadNewestRunEnd` gains rows and the `Wait` cleanup table gains an assertion.
3. **`readAgentActivity` reports a live wait for a registered `RoleGate` or fixer-role process.**
   - Registration sites. The gate runner registers under `originSlug(...)`, which returns `location.WorktreeName` (`detachfront.go:857-862`, `run.go:399-408`). Fixers register through `gatefixer.Registrar` under the pair slug (`loomcli/gatefixer.go:228`, `gatecli/cardfixer.go:175`, `webstercli/fixverify.go:42`). `gateProcessLive` reads `runproc.List(..., taskLocation.WorktreeName)`, so the owner key matches for all of them.
   - `runproc.List` is the sanctioned reader under `PATTERN-run-process-owner`. It prunes records whose process is gone and leaves out zombies, so a dead pid is not a live wait. No new signalling site is added.
   - Doc comments updated: `readAgentActivity` and the `Agents` seam in `runnotice.go:104`.
   - Docs not updated, and not wrong: `dobbyengine/doc.go:139` says "no live wait" without listing sources. `loomshed/doc.go:115-116` and `loomcli/doc.go:256` describe only the operator-wait rows. `shuttleengine/doc.go:460-467` describes the shuttle wait markers, a different source. None enumerates the waits, so none is made false. Covered.
4. **Rule documents.** The diff touches no `PATTERN.md` or `pattern/` file, and the prompt's constraints section declares no PATTERN changes, so there is nothing to land. `PATTERN-run-process-owner`, `PATTERN-named-constants` (one `runEndStampFileName` constant) and `PATTERN-durable-vs-ephemeral-state` (the stamp lives under the `.lym` shuttle root, an ephemeral location) are not violated by the diff. I found no stencil or spec that describes the quiet check's sources, so there is no stencil wording to compare. No commit carries a `Pattern-Departure:` trailer, and none is needed. The commit message claims (stamp at the run-directory root, pane process of a cold fixer, detached gate runner) match the code I read.

Tool calls (6 of 6):
1. `grep` of doc.go, `*.md` and `pattern/` for live-wait and quiet-check descriptions: my context held no list of docs that name the wait sources.
2. `grep` of every `os.ReadDir(root)` and `runDirRoot(` site: my context held only some of the root scanners.
3. Read `rundir.go` 300-390 for `sweepOrphans`: its non-directory handling and age guard were not in the manifest.
4. Read `battencli/cleanup.go` 195-265: to see whether the `ReadDir` hit at line 224 scans the run-directory root.
5. `grep -n -A6` of `ReadDir(root)` in six shuttleengine files: to check how each scan treats entries. Its output was too narrow to settle that.
6. `grep -c "IsDir()"` over the same six files: to confirm each has a directory check.
