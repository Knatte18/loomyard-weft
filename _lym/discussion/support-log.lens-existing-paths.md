```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens review of the fabric-git-free-core decision record and support log.

**Paths traced, per decision**

- **D1, layout move.**
  - `TestEnforcement_FabricVocabulary`'s bare-side owner set: survives, because `## PATTERN changes` adds the new packages to fabric.
  - `geometryTokenOwners` in `internal/lymcwd/enforcement_test.go`: breaks.
  - `weftnameImportOwners`: breaks.
  - PATTERN backgrounds that cite `fabricengine.` symbols: break (stale).
- **D1, leaf import set.** `weftnameImportOwners`: breaks (see finding 1).
- **D2, snapshot under the verb's lock.**
  - In-process verbs: survive.
  - Detached teardown child, `SpawnPairPush` and the runpusher background pusher: break, and the record says nothing about them (finding 2).
- **D3, loop in the engine.**
  - In-process push retry: survives.
  - Retries inside a detached child after the verb returned: unaddressed (finding 2).
- **D4, Mutations and destroy.go.**
  - The `PATTERN-fabric-boundary` rule "`rec *Mutations` threaded into `destroy.go` only": breaks (finding 3).
  - Destroy's execution-time checks: survive.
- **D7, retarget.** The `testonlycode` allowlist and `tierpurity` scans name `fabricengine` paths and symbols: they may go stale (LOW, folded into finding 1).
- **D12, import guard.**
  - `gitkit.HermeticGitEnv` and the tmux kit's `Main` are allowed: survives.
  - Config reaching tests as a written config file: survives.
  - `configengine` reached transitively through `gitkit` or `hubforge` is no direct import: survives.

**Findings**

1. **MEDIUM, D1/D7 and `## PATTERN changes`:** moving path constructors and names out of `internal/fabricengine` breaks two existing `lymcwd` guard maps and stales PATTERN text. The record is silent on all three.
   - **Geometry tokens.** `geometryTokenOwners` lets only the directory `internal/fabricengine` declare or construct `-LYMHUB` (its `HubPath` constructor), `_portals` and `_launchers`. If `HubPath` and the portal or launcher paths move to `internal/fabriclayout`, `TestEnforcement_FabricVocabulary` fails.
   - **weftname imports.** `weftnameImportOwners` allows only {fabricengine, fabriccli, gitkit, weftname, hubforge, lymcwd} to import `internal/weftname`. Layout's worktree and records paths and the core's records layout need weftname-derived names.
   - **Stale PATTERN text.** `PATTERN-lym-files` names `fabricengine.structuralCommittedDirs` and `structuralNeverCommittedDirs`. `PATTERN-cwd-resolution` says only `fabricengine` writes or git-reads the anchor marker. `PATTERN-fabric-boundary`'s background names `fabricengine.ScopedPathspec`, `fabricengine.mutateGitExclude` and `classifyPaths`, all of which D1 and the Notes put into the core or the layout.
   - **Fix:** state that both owner maps gain the new packages, and list `PATTERN-lym-files`, and `PATTERN-cwd-resolution` if its wording changes, under `## PATTERN changes`.
   - Evidence: `internal/lymcwd/enforcement_test.go` (`geometryTokenOwners`, `weftnameImportOwners`), `pattern/PATTERN-lym-files.md` and `pattern/PATTERN-cwd-resolution.md`.
2. **MEDIUM, D2/D3/D4:** part of fabric runs in detached child processes, and the record never says where the snapshot and the core's loop run there. Examples:
   - `Remove` and `Cleanup`'s child pushes archive tags, then the branch, then deletes on origin, re-pushing leftover local `archive/<slug>/*` tags first.
   - `Add` callers push through `SpawnPairPush`.
   - The runpusher background pusher brings commits to origin.

   D2's "read under the lock the verb takes, at the point it takes it" and D3's "engine runs it and asks the core again" assume one in-process verb. The child acts after the verb returned, under the teardown or remote-refs lock, against origin state the parent's snapshot never held.
   - **Risk:** if the action list for teardown is computed in the parent and replayed in the child, the "tag before delete on origin" ordering rests on a stale snapshot.
   - **Fix:** state that a detached child reads its own snapshot under its own lock and asks the core itself, or that its action list is fixed in the parent and why that is safe.
   - Evidence: `PATTERN-fabric-boundary` "Unwire and teardown"; detached-spawn sites in `add.go`, `cleanup.go`, `coalesce.go`, `commit.go` and `destroy.go`.
3. **MEDIUM, D4:** "the executor records each action it ran" in `fabricengine.Mutations` contradicts the existing rule that the `rec *Mutations` recorder is threaded into `destroy.go` only. Neither the record nor `## PATTERN changes` changes that rule.
   - **Fix:** confine the recorder to the destructive executor in `destroy.go`, or add the change to `PATTERN-fabric-boundary`.
   - Evidence: `PATTERN-fabric-boundary` "Destruction chokepoint", last bullet.
4. **LOW, D7/D12:** existing scans that name `fabricengine` paths or symbols go stale as names move. These include the `testonlycode` allowlist and the `tierpurity` banned-helper list, which is keyed on `hubforge`. The guard at the vocab scan's `refscanner.go` row is one example. This is not a call-site list: the rule the record should state is that each part moves a scan's entry in the same change that moves its symbol.
   - Evidence: `internal/lymcwd/enforcement_test.go` has a key for `internal/fabricengine/refscanner.go`.

**Tool calls**

- grep for detached spawns in fabricengine and for the owner maps in `lymcwd/enforcement_test.go`: my context lacked which files spawn detached children and how the vocabulary scan's owners are keyed.
- read of the `geometryTokenOwners` and `weftnameImportOwners` spans, plus a grep of fabricengine mentions in the `PATTERN-cwd-resolution` and `PATTERN-lym-files` text: my context held only the PATTERN index lines, not these guard maps or the entries' wording about fabricengine.
