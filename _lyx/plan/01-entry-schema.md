# Card 1 — entry-schema

**Edit:**
- `internal/boardengine/task.go`
- `internal/boardengine/store.go`
- `internal/boardengine/layer.go`
- `internal/boardengine/render.go`
- `internal/boardengine/board.go`
- `internal/battencli/wire.go`
- `internal/battenshed/deps.go`
- `internal/boardengine/task_test.go`
- `internal/boardengine/store_test.go`
- `internal/boardengine/layer_test.go`
- `internal/boardengine/render_test.go`
- `internal/boardengine/board_test.go`
- `internal/battencli/lifecycle_integration_test.go`

**Intent:**
Give every board entry a `tier` and a `type`, rename the recipe field to `recipe`, retire `deferred`, and replace the deferred dependency rules with the tier rule (Decisions 2–7).
The record keeps the Go name `Task` so the `{"task": ...}` envelope and every external caller (`GetTask`, `SetStatus`) stay unchanged.

`task.go`:
- `Task` fields become `id`, `slug`, `title`, `tier`, `type`, `recipe` (omitempty), `depends_on`, `isolated`, `brief`, `body`, `status` (omitempty), `short_name` (omitempty), in that JSON order.
  The old `Type` field (recipe name) becomes `Recipe` with JSON key `recipe`; `Type` now holds the entry kind; `Deferred` is deleted.
- Declare the tier bounds (1–3), the default tier 3, the closed type set `feature`, `bug`, `chore`, `design`, and the default type `feature` once, as named constants or a set in this file.
- `NewTask` applies the defaults to a field map that omits `tier` or `type`.
- One validation helper checks a built `Task`: `tier` outside 1–3 is refused naming the range;
  a `type` outside the set is refused naming the allowed set and saying a recipe name belongs in `recipe`.
  `NewTask` and `ApplyPatch` both run it, so `upsert`, `upsert-batch` and `merge` all reach it.

`store.go`:
- `upsertAllowedKeys` gains `tier`, `type`, `recipe` and loses `deferred`.
  A payload carrying `deferred` is refused with a message naming `tier: 3` as its replacement, ahead of the generic unknown-field message (the same special-case shape as the existing `phase` hint).
- `validateWrite` drops both deferred rules and gains the tier rule in both directions over the projected snapshot.
  The rule is one predicate per edge: a dependent's tier must be greater than or equal to its dependency's tier, compared only when the dependency is not `done`;
  the dependent's own status never exempts the edge.
  So every non-`done` dependency of the incoming entry must have a tier less than or equal to its own,
  and every entry depending on the incoming entry, `done` or not, must have a tier greater than or equal to the incoming entry's tier, unless the incoming entry is itself `done`.
  The refusal names the offending dependency and which entry to promote or demote.
  Dangling, isolated and cycle checks stay as they are.
- `SetStatus` to a value other than `done` runs the tier rule for that entry, so reopening a done dependency cannot leave a lower-tier dependent behind it;
  setting `done` stays unchecked, since it never introduces a violation and `MarkTaskDone` must not fail on unrelated legacy data.
- `BriefTask` drops `deferred` and gains `tier` and `type`.

`layer.go`: `ComputeLayers` drops the `__deferred__` bucket and `RenderOrder` its rank; `__done__` and the isolated `Z` bucket stay.
`render.go`: drop the `__deferred__` cases from `bucketHeader` and `isSpecialBucket`; the README shape itself is rewritten in card 6.
`board.go`: `taskToUpsertFields` emits `tier`, `type` and `recipe` instead of `deferred` and the old `type`.

`internal/battencli/wire.go`'s `ReadBoardType` returns `task.Recipe`, and the field comment on `ReadBoardType` in `internal/battenshed/deps.go` names the entry's `recipe` field; the seam keeps its name.
`internal/battencli/lifecycle_integration_test.go`'s `seedBoardTask` seeds the recipe through `"recipe"`.

Tests, in the existing test files: defaults (tier 3, type `feature`) through `UpsertTask` and `UpsertTasksBatch`;
`tier` 0 and 4 refused; an unknown `type` refused with a message naming the set and `recipe`; `deferred` refused naming `tier: 3`;
a dependency on a higher-tier entry refused on create, on demotion of the dependency, and on promoting the dependent to a lower tier number via upsert;
a dependency on a `done` entry accepted whatever its tier; demoting a dependency of a `done` dependent to a higher tier refused; `SetStatus` reopening a done higher-tier dependency refused.
Delete the deferred-rule and deferred-bucket cases, and repoint the `type` round-trip tests (`TestPromoteNotePreservesType` and the `task_test.go`/`store_test.go` cases) at `recipe`.

**ImpactSummary:** Field rename ripples to `battencli`'s recipe read and every boardengine test touching `deferred` or `type`; `boardcli` help text still lists `deferred` until card 8.
