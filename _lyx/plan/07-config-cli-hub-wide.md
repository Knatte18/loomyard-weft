# Card 7 — `lyx config` edits a hub-wide module at the board dir and commits it in `_board`

**Create:**
- `internal/configcli/hubwide.go`

**Edit:**
- `internal/configcli/configcli.go`
- `internal/configcli/menu.go`
- `internal/configcli/doc.go`
- `internal/configcli/configcli_test.go`
- `internal/configcli/configcli_integration_test.go`
- `PATTERN.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/fabricengine#Bolt.CommitWritten`
- `plan:internal/configengine#EditPath`
- `plan:internal/configengine#StagingFile`
- `internal/configreg#Module`
- `internal/configengine#Set`
- `internal/configengine#Load`
- `internal/fabricengine#Bolt.Push`

**Intent:**
`lyx config <module>`, `--set`, `--print` (one module and the all-modules dump) and `lyx config menu` address a hub-wide module (`configreg.Module.HubWide`) at `configengine.ConfigFile(fabricengine.BoardDir(hub), name)` and commit its edits in `_board` (D5).
Per-worktree modules keep today's behaviour and fabric sync.

Wiring in `configcli.go`:
- Replace the single `baseDirOf(l)` with a small dirs value holding the worktree base (`baseDirOf(l)`) and the board dir (`fabricengine.BoardDir(l.HubPath)`).
  `dispatch`, `printModule`, `printAll` and `menu` take it, and pick each module's base from the registry field, never from a module name.
- Add a hub-commit seam beside `syncFunc`: `func(module string, write func() error) error`.
  `resolveReal` builds the real one over `fabricengine.NewBolt(boardDir).CommitWritten`, whose `write` runs the given closure and returns `configengine.ConfigFileRel(module)`, then `Bolt.Push`, both with `fabricengine.EnvSyncOptions()`.
  Unit tests pass a fake that runs `write` and returns a chosen error.

Hub-wide write paths, in `hubwide.go`:
- `--set`: inside the commit seam's `write`, snapshot the hub file (or its absence), run `configengine.Set` at the board dir, then validate with the strict `configengine.Load(boardDir, module, template, openMaps...)`.
  On a `Set` or validation error, restore the snapshot (or remove a file that was absent) and return the error.
- Edit: outside the lock, copy the hub file's bytes (the template when absent) to `configengine.StagingFile(<worktree base>, module)` and run `configengine.EditPath` on it, so the editor never holds the board write lock.
  Then, inside `write`: refuse if the hub file no longer matches the bytes the staging copy was taken from (a concurrent edit), and otherwise write the staged bytes to the hub file atomically, validate with the strict load, and restore the snapshot on failure.
  The staging file is removed on every exit.
- The caller tells a refused write from a failed commit by recording the `write` closure's own error.

Messages name the file actually edited, its absolute hub path for a hub-wide module:
- success: `edited and committed <path> in _board`;
- refused edit or set: the validation error plus `<path> unchanged`;
- editor abort: `aborted: <path> unchanged`;
- failed commit or push: `edited <path> but it is uncommitted: <err>; the next board sync commits it`, with the file left on disk.
The per-worktree messages keep their `_lyx/config/<module>.yaml` wording.

Bound: any worktree can change the hub's board vocabulary through `lyx config board`; that is the trust boundary `lyx board upsert` already has.
The strict load before commit bounds a malformed edit to a refused write that never reaches `_board`, and the commit pathspec is the one module file.
This also fixes `lyx config fabric`, which today edits a per-worktree `fabric.yaml` nothing reads.

Docs and the invariant in this commit (D7):
- `buildConfigLong` and `configcli/doc.go` say hub-wide modules (named from the registry, not a literal list) are edited at the hub and committed in `_board`, and per-worktree modules sync fabric.
- `PATTERN.md` gains, under "Paths and geometry": ``- `PATTERN-hub-wide-config` — Adding or reading a config module that describes a hub-level fact: it is marked `HubWide` in `configreg` and read and written only at `<BoardDir>/_lyx/config/`, never in a worktree's `_lyx/config/`.`` with no background file.
- `docs/overview.md`: the paragraph whose example writes `_lyx/config/board.yaml` through the junction uses a per-worktree module instead.
  It also adds one sentence beside the repo-wide `fabric.yaml` paragraph saying `board.yaml` is hub-wide like it.

Test migration in `configcli_test.go`: tests that used `board` as a generic per-worktree module move to a per-worktree module such as `loom`.
The open-map `--set` tests (`TestDispatchSet_OpenMapAddsLabel`, `TestDispatchSet_OpenMapRefusesListShape`) keep `board` and now exercise the hub path through the fake commit seam.
`TestMenuStatus` seeds the board dir for its hub-wide expectation.

Coverage:
- `configcli_test.go`: one table-driven test over `board` through `dispatch`, with the worktree and board dirs as separate temp dirs. Rows:
  - `--set labels.x=desc` writes the board-dir file, leaves the worktree file absent, calls the commit seam and never `syncFunc`;
  - `--print board` prints the board-dir file;
  - a `--set` the strict load rejects (a list where the template has a scalar) is refused, and the board-dir file is byte-identical afterwards;
  - a commit-seam error leaves the edited file on disk, and the error names the hub path and "uncommitted";
  - an editor edit writes the staged bytes to the board-dir file;
  - an editor edit the strict load rejects leaves the board-dir file unchanged, and no staging file remains.
- `configcli_integration_test.go` (integration, `hubforge` hub with one pair): `lyx config board --set labels.x=desc` run from the pair (`RunCLIIn`) changes the hub `board.yaml`, leaves `_board` clean with HEAD's commit touching that file, and writes no per-worktree `board.yaml`.
  A second row does the same for `lyx config fabric --set branch_prefix=x`.

**ImpactSummary:** `configcli`'s dispatch, print, menu and edit/set paths gain a hub-wide branch and a commit seam; per-worktree behaviour is unchanged apart from the test-module rename, and PATTERN and overview docs gain the hub-wide rule.

**Commit:** 7: config: edit hub-wide modules at the board dir and commit them in _board
