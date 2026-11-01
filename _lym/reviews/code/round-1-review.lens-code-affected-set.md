```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set. For each change the diff makes, the set it forces outside the touched lines, and whether the diff covers it.

**Per-change list**

- **`_board` literal declared once (D9).**
  - Go code: covered. The only non-test literal is `internal/lymcwd/anchor.go:25`. The `TestEnforcement_GeometryLiterals` owner row names `internal/lymcwd` alone (`internal/lymcwd/enforcement_test.go:224`).
  - `livestate_doc_test.go`: no `_board` left.
  - Stencils, `plugins/`: no `_board` found.
  - Specs: this checkout carries none, so I could not check them.
  - Not covered:
    - `docs/overview.md` (see finding 1).
    - `tools/sandbox/SANDBOX-*.md` and `README.md`, which still hold `_board` (see finding 2).
- **`BoardDir` to `BoltDir` split.** Covered. The remaining `fabricengine.BoardDir` callers all read or write board-own data: `board.json`, `ReadOwners`, `ReadTasks`, `boardengine.New` and `cfg.Path`. I read the context at `fabriccli/anchordeps.go:70`, `ideengine/menu.go:30`, `hubgeom/dobbygeom.go:87`, `adoptcli/deps.go:58`, `beamcli/deps.go:33`, `battencli/cancelcmd.go:174`, `battencli/owner.go:43` and `orchcli/ownerpause.go:45`. Where a Bolt path is needed, `BoltDir` sits beside `BoardDir` in the same function.
- **Old board replay seam removed.** Covered. A grep for `ErrBoardReplayMissing`, `InstallBoardReplay`, `fabricengine.BoardReplay`, `boardFileName`, `BoardReplayRequest` and `BoardReplayResult` finds nothing outside `_lym`.
- **Declared PATTERN changes.**
  - `PATTERN-durable-vs-ephemeral-state`: `PATTERN.md` carries the declared text, and the background file gains the root-level bullet.
  - `PATTERN-heddle-instructions`: the `PATTERN.md` line matches the declared text, and the background file is new.
  - `PATTERN-leaf-packages`: the `PATTERN.md` overview line is unchanged, as declared. The background file adds `internal/lymcwd` to `runproc`'s import set (`pattern/PATTERN-leaf-packages.md:84-85`), and `internal/runproc/leaf_enforcement_test.go:4,17` agrees.
  - All three declared entries landed.
- **`warp_exclude` key, seam, `lym heddle` verbs.** The diff adds no `docs/`, README or plugin skill text for them (see finding 3). `contracts/stencils` and `plugins/ly/skills` changed only for the `_board` sweep.
- **Commit messages and departures.** The range has no `Pattern-Departure:` trailer. Card 13 and the plan's "Shared Decisions" depart from D7's wording, but the parent amendment in the decision record covers that, so it is not a finding.

**Findings**

1. **LOW, consistency.** `docs/overview.md:172` still describes the `pathspec` list as read "at `<BoardDir>/_lym/config/fabric.yaml` … via `fabricengine.BoardDir`". Fabric's config now lives in the Bolt checkout and `ConfigDirs` takes it from `BoltDir` (Notes for the plan writer). Lines 119, 126, 143, 182 and 370 of the same file still spell `_board`. D9 requires specs, stencils and skills to be clear of the literal; it does not name `docs/`, so this is a stale-description finding, not a D9 violation.
2. **NIT, consistency.** `tools/sandbox/SANDBOX-CORE-SUITE.md` (lines 156 and 234), `tools/sandbox/SANDBOX-FABRIC-SUITE.md` (lines 117, 215, 231, 249, 265, 340, 546 and 562), `docs/sandbox-hub.md:31` and `README.md:41` keep literal `_board`. These are operator-facing, hand-run material outside D9's list, so I am reporting them only because the "no literal left" sweep reads wider than the diff.
3. **LOW, consistency.** The diff documents none of the user-visible surfaces it adds:
   - the `warp_exclude` fabric key;
   - the `lym heddle` module;
   - the worktree seam;
   - the replayer registry.

   A search of `docs/*.md`, `README.md` and `plugins` for `warp_exclude` and `heddle` returns nothing. Rubric: "the docs it names for the kind of change the diff makes land in the same diff." I found no repo rule in my context that names a doc for this kind of change, so keep this at LOW unless the handler finds one.

No BLOCKING or MEDIUM finding in this lens.

**Tool calls (3 of 6)**
1. Diff of `pattern/` and `PATTERN.md`, a repo-wide `_board` grep and an old-name grep: the manifest had the PATTERN diff only in truncated form and had no literal or old-name search.
2. Grep of `BoardDir(` and `BoardDirName` callers, plus `heddle` docs and `lymcwd` in `runproc`: to find callers that should have moved to `BoltDir`.
3. Context around eight `fabricengine.BoardDir` call sites and a grep for `warp_exclude` and `heddle` in docs: to judge whether each use is board-own and whether docs mention the new surfaces.
