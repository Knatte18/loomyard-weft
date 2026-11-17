```yaml
lens: bound-breaker
outcome: reported
```

Findings, bound-breaker lens: I attacked every claim and bound the record states.

**Findings**

1. **MEDIUM, D5.** The claim that both openers hand the file to the desktop and return does not always hold.
   - Counterexample: with no desktop detected, `xdg-open` falls back to its generic mode, for example over ssh or with `XDG_CURRENT_DESKTOP` unset. In that mode it runs `$BROWSER` or a found browser in the foreground. With a terminal browser such as `www-browser` or `lynx` it blocks until that browser exits.
   - D5 also requires the spawn to be waited on (`PATTERN-child-processes`), so `lym board open` and the VS Code task hang for that time.
   - The record states no bound for this case. Either state it as a bound, or give the wait a limit.
2. **MEDIUM, D6.** The bound names only the task-pair window as one where `alt+shift+0` or `alt+shift+9` finds no task. Three other states break the same promise:
   - `ideengine.writeVSCodeConfig` skips a worktree whose `.vscode/tasks.json` is git-tracked. That window never gets `board open`, yet the user-global block binds `alt+shift+9` to it.
   - The block is user-global and any seed rewrites it. Every window seeded before the upgrade keeps its old `tasks.json` until it is seeded again, so both keys report a missing task there.
   - In `seed`, an unresolved prime makes `isPrime` false and sends the prime down the task branch. So the prime itself loses `beam start`, not only "a beam whose origin record is unreadable".
3. **LOW, D3.** The promise that both files land in the same commit as `README.md` and stay current with it fails across lym versions.
   - The render manifest `.board-rendered.json` is gitignored and local to each machine. A machine still on an older lym writes `README.md` and leaves `board-data.js` untouched, so the commit carries stale viewer data until a newer machine writes.
   - A downgrade on the same machine is worse. The newer manifest lists `board.html` and `board-data.js`, the older binary does not produce them, so `RenderToDisk` removes both. `commitDirty`, which stages the whole tree through `Bolt.Commit`, then commits the deletion.
   - The record is silent on both cases.
4. **LOW, D5.** "Resolves the hub's Bolt checkout the way the other `lym board` verbs do" means `lymcwd.Resolve` (`boardcli/cli.go`, around line 146). That refuses unless cwd is the worktree's anchor root.
   - The VS Code task runs at the anchor, so the task works.
   - From a terminal in a subdirectory, the verb refuses. The rationale "the same verb serves a terminal" does not hold there.
   - This is a read-only verb, which `PATTERN-cwd-resolution` lets use `ResolveWithin`.
5. **LOW, D7.** "The README already wraps each project in `<details open>`, so the GitHub app renders the element" is an inference, not evidence. Nothing shows GitHub Mobile's file view expanding or collapsing `<details>`. The phone goal rests on this fact, and the record leaves it unverified.
6. **LOW, D5.** The way forward `lym board rerender` carries `AudienceInternal`, and `Board.Rerender` renders without a sync.
   - After it, `board.html` exists locally, so `open` works.
   - But the files stay uncommitted until the next board write.
   - Either name this in the refusal or accept it.

**Claims checked**

- D1, HTML escaping: holds. `json.Marshal`, `MarshalIndent` and the default `Encoder` all escape `<`, `>`, `&`, U+2028 and U+2029. An acceptance criterion pins it.
- D1, type and area split by vocabulary: holds. `Vocabulary.IsType` exists.
- D2, identical bytes leave no git change: holds.
  - `Bolt.Commit` stages only real diffs.
  - `renderOwnerWrite` lists every rendered name, and `CommitWritten`'s `stageablePaths` commits only changed ones.
- D3, owner-set path commits the names `Render` returns: holds. `renderOwnerWrite` iterates `Render`'s map and adds stale names.
- D3, replay: holds. `Claims` routes the files into `Apply`, which ignores paths that are not the store, design, record or raddle files, then re-renders through `RenderToDisk` (`replay.go:472`).
  - Adding the names to `Claims` (or to `isBoardOverview`) suffices.
  - Neither name matches `boardDesignSlug` or `boardRecordSlug`.
- D3, stale removal by manifest: holds on one version. It breaks across versions, as in finding 3.
- D4, missing-data line when `window.BOARD` is undefined: holds.
- D5, no argument, opens only the hub's `board.html`: holds.
- D5, refusal starts nothing when `board.html` is absent: holds.
- D5, task cwd resolves: holds. It runs at the anchor, as `reed attach` does today.
- D6, prime and beam classification reused from `seed`: partly broken, as in finding 2.
- D6, mill keys avoided: holds.
- D7, tables inside `<details>`: holds. `collapsibleProjectSection` already renders tables inside `<details>`.

**Tool calls**

- `replay.go` grep and `renderOwnerWrite` body: my context had the claim but not the owner-write and replay-apply code.
- `boardcli` cwd resolution and `commitDirty`: needed to check D5's "the way other board verbs do" and D3's commit scope.
- `replay.go` lines 140–165 and 345–365: needed to see how `Apply` routes claimed non-entry files.
