# Card 13 — fabrickit-snapshot-builder

**Create:**
- `internal/testkit/fabrickit/doc.go`
- `internal/testkit/fabrickit#Builder`
- `internal/testkit/fabrickit#NewSnapshot`
- `internal/testkit/fabrickit#Builder.Phases`
- `internal/testkit/fabrickit#Builder.LocalRef`
- `internal/testkit/fabrickit#Builder.OriginRef`
- `internal/testkit/fabrickit#Builder.Worktree`
- `internal/testkit/fabrickit#Builder.Anchor`
- `internal/testkit/fabrickit#Builder.RecordsStore`
- `internal/testkit/fabrickit#Builder.PendingTeardown`
- `internal/testkit/fabrickit#Builder.Outcome`
- `internal/testkit/fabrickit#Builder.ReadFailure`
- `internal/testkit/fabrickit#Builder.Build`
- `internal/testkit/fabrickit_test#TestMain`
- `internal/testkit/fabrickit_test#TestBuilder`
- `internal/testkit#TestEnforcement_FabrickitLeafOnly`

**Edit:**
- `cmd/lym#testOnlyCodeAllowed`

**Uses:**
- `internal/fabriccore/doc.go`
- `internal/testkit/doc.go`
- `internal/testkit#TestEnforcement_PermkitStdlibOnly`
- `internal/testkit/scankit#AssertImportAllowlistNoStale`

**Intent:** Add `internal/testkit/fabrickit` (D8): the one builder of made-up snapshots the tests of `internal/fabriccore` and `internal/fabricengine` share.
It imports only the standard library, `internal/fabriclayout` and `internal/fabriccore`, never `gitexec`, `gitkit` or `internal/hubforge`.
It builds fabric's own state only: refs, worktrees, anchors, the records store, pending teardowns, outcomes and read failures, and a seam fact it sets later is a value in fabric's vocabulary.

- `NewSnapshot` starts an empty builder; each setter records its value and returns the builder, so a row reads as one chain.
  `LocalRef` and `OriginRef` set one ref of a store, `OriginRef` also marking the store's origin as read; `Worktree`, `PendingTeardown`, `Outcome` and `ReadFailure` append in call order; `Anchor` sets one worktree's marker; `Phases` and `RecordsStore` set their field.
- `Build` returns a `fabriccore.Snapshot` that shares no map or slice with the builder, so a later setter call never changes a snapshot already built.
- `doc.go` says what the kit builds, its import set and that it stands in for no other module's data.

Tests, tier0:
- `TestBuilder` (table) builds snapshots and compares each with a literal `fabriccore.Snapshot`, in two rows:
  one chain that calls every setter, so a setter that drops or misplaces its value fails it;
  and one that calls every setter again after `Build` and checks the built snapshot is unchanged, then overwrites the first element of each of the built snapshot's slices and wants the next `Build` unchanged by it.
  The setters after `Build` catch a shared map, and the overwrite a shared slice, which a later append never shows.
- `TestEnforcement_FabrickitLeafOnly` (`//lym:guard`, `//lym:watch internal/testkit/fabrickit/*.go internal/testkit/scankit/*.go`), beside the permkit guard in `internal/testkit`'s enforcement test, calls `AssertImportAllowlistNoStale` with `internal/fabriclayout` and `internal/fabriccore`.
  `TestEnforcement_TestkitInvariant` already bars a kit's `os/exec`, `gitexec`, `gitkit`, `hubforge` and spawning imports; this test holds the narrower closed set D8 names.
- `TestMain` runs under `tmuxkit.Main`.

`fabriccore.Refusal.Error` has no production caller until part 2, `fabric-core-push-origin`, and from this card on the kit, a test-support package, brings `internal/fabriccore` into the test-only-code analysis.
So `cmd/lym#testOnlyCodeAllowed` admits `internal/fabriccore.Refusal.Error` here, as the added entry "A chain part may leave production code that only a named later part first calls" allows:
its reason names `TestRefusal_Error` and `fabric-core-push-origin` as the part that adds its first production caller and removes the entry.

**ImpactSummary:** One allowlist entry in the test-only-code guard, for `fabriccore.Refusal.Error`, which the kit's import first brings into the analysis; no production behavior changes.

**Commit:** `13: fabrickit-snapshot-builder`
