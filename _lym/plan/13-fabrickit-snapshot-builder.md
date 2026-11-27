# Card 13 — fabrickit-snapshot-builder

**Create:**
- `internal/testkit/fabrickit/doc.go`
- `plan:internal/testkit/fabrickit#Builder` -> `type Builder struct`
- `plan:internal/testkit/fabrickit#NewSnapshot` -> `func NewSnapshot() *Builder`
- `plan:internal/testkit/fabrickit#Builder.Phases` -> `func (b *Builder) Phases(phases ...fabriccore.Phase) *Builder`
- `plan:internal/testkit/fabrickit#Builder.LocalRef` -> `func (b *Builder) LocalRef(store fabriccore.Store, ref, sha string) *Builder`
- `plan:internal/testkit/fabrickit#Builder.OriginRef` -> `func (b *Builder) OriginRef(store fabriccore.Store, ref, sha string) *Builder`
- `plan:internal/testkit/fabrickit#Builder.Worktree` -> `func (b *Builder) Worktree(entry fabriclayout.WorktreeEntry) *Builder`
- `plan:internal/testkit/fabrickit#Builder.Anchor` -> `func (b *Builder) Anchor(worktree, anchor string) *Builder`
- `plan:internal/testkit/fabrickit#Builder.RecordsStore` -> `func (b *Builder) RecordsStore(store fabriccore.RecordsStore) *Builder`
- `plan:internal/testkit/fabrickit#Builder.PendingTeardown` -> `func (b *Builder) PendingTeardown(record fabriclayout.PendingTeardown) *Builder`
- `plan:internal/testkit/fabrickit#Builder.BackoffDelays` -> `func (b *Builder) BackoffDelays(delays ...time.Duration) *Builder`
- `plan:internal/testkit/fabrickit#Builder.Outcome` -> `func (b *Builder) Outcome(outcome fabriccore.Outcome) *Builder`
- `plan:internal/testkit/fabrickit#Builder.ReadFailure` -> `func (b *Builder) ReadFailure(failure fabriccore.ReadFailure) *Builder`
- `plan:internal/testkit/fabrickit#Builder.Build` -> `func (b *Builder) Build() fabriccore.Snapshot`
- `plan:internal/testkit/fabrickit#TestMain` -> `func TestMain(m *testing.M)`
- `plan:internal/testkit/fabrickit#TestBuilder` -> `func TestBuilder(t *testing.T)`
- `plan:internal/testkit#TestEnforcement_FabrickitLeafOnly` -> `func TestEnforcement_FabrickitLeafOnly(t *testing.T)`

**Uses:**
- `internal/fabriccore/doc.go`
- `internal/testkit/doc.go`
- `internal/testkit#TestEnforcement_PermkitStdlibOnly`
- `internal/testkit/scankit#AssertImportAllowlistNoStale`

**Intent:** Add `internal/testkit/fabrickit` (D8): the one builder of made-up snapshots the tests of `internal/fabriccore` and `internal/fabricengine` share.
It imports only the standard library, `internal/fabriclayout` and `internal/fabriccore`, never `gitexec`, `gitkit` or `internal/hubforge`.
It builds fabric's own state only: refs, worktrees, anchors, the records store, pending teardowns, backoff delays and outcomes, and a seam fact it sets later is a value in fabric's vocabulary.

- `NewSnapshot` starts an empty builder; each setter records its value and returns the builder, so a row reads as one chain.
  `LocalRef` and `OriginRef` set one ref of a store, `OriginRef` also marking the store's origin as read; `Worktree`, `PendingTeardown`, `Outcome` and `ReadFailure` append in call order; `Anchor` sets one worktree's marker; `Phases`, `RecordsStore` and `BackoffDelays` set their field.
- `Build` returns a `fabriccore.Snapshot` that shares no map or slice with the builder, so a later setter call never changes a snapshot already built.
- `doc.go` says what the kit builds, its import set and that it stands in for no other module's data.

Tests, tier0:
- `TestBuilder` (table) builds snapshots and compares each with a literal `fabriccore.Snapshot`: one row per setter, one for an empty builder, and one that calls a setter after `Build` and checks the built snapshot is unchanged.
- `TestEnforcement_FabrickitLeafOnly` (`//lym:guard`, `//lym:watch internal/testkit/fabrickit/*.go internal/testkit/scankit/*.go`), beside the permkit guard in `internal/testkit`'s enforcement test, calls `AssertImportAllowlistNoStale` with `internal/fabriclayout` and `internal/fabriccore`.
  `TestEnforcement_TestkitInvariant` already bars a kit's `os/exec`, `gitkit` and spawning imports.
- `TestMain` runs under `tmuxkit.Main`.

**Commit:** `13: fabrickit-snapshot-builder`
