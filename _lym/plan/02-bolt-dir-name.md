# Card 2 — bolt-dir-name

**Create:**
- `plan:internal/lymcwd#BoltDirName` -> `const BoltDirName = "_board"`
- `plan:internal/fabricengine#BoltDirName` -> `const BoltDirName = lymcwd.BoltDirName`
- `plan:internal/fabricengine#BoltDir` -> `func BoltDir(hub string) string`

**Delete:**
- `internal/lymcwd#boardDirName`

**Edit:**
- `internal/lymcwd#boardDir`
- `internal/lymcwd#readRecordedAnchor`
- `internal/lymcwd#TestEnforcement_GeometryLiterals`
- `internal/runproc#HubOwner`
- `internal/runproc/doc.go`
- `internal/fabricengine#BoardDirName`
- `internal/fabricengine#BoardWriteLockPath`
- `internal/fabricengine#StencilsDir`
- `internal/fabricengine#SpecsDir`
- `internal/fabricengine#HubScratchDir`
- `internal/fabricengine#AdoptScratchDir`
- `internal/fabricengine#HubLogsDir`
- `internal/fabricengine#LogArchiveDir`
- `internal/fabricengine#RunProcDir`
- `internal/fabricengine#HubReservedNames`
- `internal/fabricengine#filterHubReserved`
- `internal/fabricengine#ConfigDirs`
- `internal/fabricengine#TestBoardDir`

**Intent:** Declare the Bolt directory name once (D9), so `bolt-dir-rename` later changes one constant's value.

- `internal/lymcwd` declares the exported `BoltDirName = "_board"`, the one Go declaration of the literal; its doc comment says it names the Bolt checkout inside a hub and why it lives here: `lymcwd` reads the hub-wide anchor marker in the Bolt checkout, and `fabricengine` and `runproc` cannot be imported by `lymcwd`.
  The private `boardDirName` goes; `boardDir` and `readRecordedAnchor`'s stale-marker message take `BoltDirName`, and every comment in `anchor.go` names "the Bolt checkout", never the path.
- `runproc.HubOwner` is set from `lymcwd.BoltDirName`; `runproc` gains the `lymcwd` import, which closes no cycle since `lymcwd` imports only the standard library, `internal/dotgit` and `internal/weftname` (card 1 records it in PATTERN-leaf-packages's background).
  `internal/runproc/doc.go` says `HubOwner` is the Bolt checkout's directory name, without spelling the path.
- `internal/fabricengine` re-exports it as `BoltDirName`, the name every package outside `lymcwd` and `runproc` uses, and adds `BoltDir(hub)`, `filepath.Join(hub, BoltDirName)`, the Bolt checkout's path.
  `BoardDirName` is set from `BoltDirName` and `BoardDir` stays built from `BoardDirName`; their comments say `BoardDir` names the board's own data (`board.json`, its overview and design files) and `BoltDir` the checkout everything else lives in.
- What lives in Bolt beside the board takes its place from `BoltDir`/`BoltDirName`: `BoardWriteLockPath` (the Bolt write lock), `StencilsDir`, `SpecsDir`, `HubScratchDir` and the scratch paths built on it (their comments name "the Bolt checkout's `.lym`" and the like, no `<hub>/_board` path), `HubReservedNames` (whose comment and `filterHubReserved`'s name the Bolt directory, not `_board`), and `ConfigDirs`, whose `Board` field is now `BoltDir(l.HubPath)`.
  Every value is unchanged, since both names hold one string.
- `TestEnforcement_GeometryLiterals`'s owner row for `_board` narrows to `{"internal/lymcwd"}`, and its comment drops the dual-owner explanation for that token; its policed-token list and planted fixture sources keep spelling the token they police.
  The scan passes because `fabricengine` and `runproc` now hold no `_board` string in a path-construction context.
- `TestBoardDir` gains a row asserting `BoltDir(hub)` equals `filepath.Join(hub, lymcwd.BoltDirName)` and equals `BoardDir(hub)`, so a later value change of the constant moves both.

**ImpactSummary:** Values unchanged; the deleted private constant has its only callers inside `anchor.go`; the geometry guard's `_board` row narrows to `internal/lymcwd`.
