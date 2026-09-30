# Card 3 — loomengine-durable-friction

**Create:**
- `plan:internal/loomengine#LoomDurableDir` -> `func LoomDurableDir(l *lyxcwd.Location) string`
- `plan:internal/loomengine#LoomDurableDirRel` -> `func LoomDurableDirRel() string`

**Edit:**
- `internal/loomengine#LoomFrictionDir`
- `internal/loomengine#LoomFrictionArchivePrefix`
- `internal/loomengine#LoomFrictionLock`
- `internal/loomcli#loomCLI.reflectFrictionRow`
- `internal/loomengine/friction_test.go`
- `cmd/lyx/notransients_test.go`

**Uses:**
- `internal/loomengine#LoomScratchDir`
- `internal/loomengine#loomDirName`
- `internal/loomengine#frictionDirName`

**Intent:**
Move friction notes onto the durable tree so they are run records that get committed, while the reflection lock stays ephemeral at the mirrored subpath (Durable-vs-Ephemeral State Invariant).

- `LoomDurableDirRel()` returns `lyxdirs.LyxDirName` joined with `loomDirName` (`_lyx/loom`), and `LoomDurableDir(l)` returns it joined onto `l.AnchorPath()`: the durable counterpart of `LoomScratchDir`.
  Both carry doc comments stating that everything under this directory is tracked run content committed by loom's per-transition status commit.
- `LoomFrictionDir` becomes `LoomDurableDir(l)` joined with `frictionDirName`; its doc comment drops the "never-tracked ephemera" rationale and says the notes are run records.
  `LoomFrictionArchivePrefix` follows it unchanged in form (`LoomFrictionDir(l) + "-"`), so archive siblings land under `_lyx/loom/` too.
- `LoomFrictionLock` stays at `.lyx/loom/friction.lock`: build it as `LoomScratchDir(l)` joined with `frictionDirName + ".lock"`, no longer derived from `LoomFrictionDir`.
  Keep the doc comment's reason the lock is not inside the directory, and add that it sits at the mirrored ephemeral subpath of the durable friction directory.
- `reflectFrictionRow`'s doc comment names `.lyx/loom/friction/`; change it to `_lyx/loom/friction/`.

Every consumer (loom, webster, plan, discussion, reflection) keeps calling the same accessors, so no call site changes.

Tests: rewrite `friction_test.go` so `LoomFrictionDir` sits under the durable `_lyx/loom` tree at a subpath anchor with its parent equal to `LoomDurableDir`, the archive prefix equals `LoomFrictionDir(l) + "-"`, `LoomFrictionLock` sits under `.lyx/loom` with its parent equal to `LoomScratchDir`, and `LoomDurableDirRel` is the anchor-relative suffix of `LoomDurableDir`.
In `cmd/lyx/notransients_test.go`, move `loomengine.LoomFrictionDir` from `transientSet` to `durableSet`, add `loomengine.LoomDurableDir` to `durableSet`, and add `loomengine.LoomFrictionLock` to `transientSet`.

**ImpactSummary:** Friction notes and their archives move from .lyx/loom to _lyx/loom for every consumer of the unchanged accessors; the reflection lock path is unchanged.
