```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness lens over the code the fix changed.

**Findings**

1. **MEDIUM**: `internal/stencilstore/skew.go`, `writtenByNewerBinary` and `refuseNewerBoardCopy`. Every read of a stencil whose recorded writer differs from the running one now calls the registered ordering before the cheap body-hash check.
   - The ordering that `cmd/lym/stencilseed.go` registers is `fabricengine.StencilSource(...).Older`. At `internal/fabricengine/warpforward.go:84-97` it runs `gitrepo` `SHAExists` twice and `IsAncestor`, which means git subprocesses.
   - `Reconcile` writes nothing for an untouched copy whose body equals the shipped default (`decide`, `StateUntouched` → `decision{}`), so that copy keeps its old writer. Under a production binary many stencils therefore have a recorded revision different from the running one.
   - Each such read (`Read`, an include, an engine block, once per render) spawns git, and only afterwards does `BodyHash(onDisk) == BodyHash(def)` return nil.
   - Running the body-hash comparison first would remove the git spawn for the common case. I have not run this; the finding comes from reading the code.

2. **LOW**: `internal/stencilstore/skew.go`, `RegisterOrdering` and `cmd/lym/stencilseed.go:seedStencils`. The ordering is registered only in `seedStencils`, which returns early when `skipStencilSeed(cmd)` is set or `stencilSeedTarget` finds no hub.
   - In those processes, including the detached per-hub daemon the comment names, the read path falls back to commit-time ordering. F4's equal-time and earlier-time descendant case then goes unchecked there.
   - This is a coverage gap in the F4 fix, not a wrong result.

**Traced, no defect**

- `Read` and `readInclude` return `*NewerBoardCopyError` directly, with no `%w` wrapping. `errors.As` works, and `errors.Is(err, os.ErrNotExist)` in `writeEngineBlocksInPlace` is false for it.
- The test rows work as intended. A flipped time comparison fails `NewerWriter*` and `EqualCommitTime`. A removed `refuseNewerBoardCopy` call fails the include, block and top-level rows. The `Writer{}` no-writer row cannot be refused.
- `SetSkewStateForTest` restores all three globals. It is safe because the test and its subtests are not parallel.
- A nil ordering from `StencilSource` with an empty revision stores nil and falls back to commit time, as documented.
- The listed non-test callers only propagate `err`. I read the call-site lines, not their bodies, so none of them is checked for swallowing the error.

**Further calls**

1. `grep` for non-test callers of `stencilstore.Read`, `ReadSet` and `ReadForEngine`: my context had no list of them.
