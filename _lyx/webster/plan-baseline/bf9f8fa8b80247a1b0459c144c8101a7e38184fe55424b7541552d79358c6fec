# Card 3 — board-kind-and-labels

**Edit:**
- `internal/boardengine/task.go`
- `internal/boardengine/store.go`
- `internal/boardengine/board.go`
- `internal/boardengine/layer.go`
- `internal/boardengine/render.go`
- `internal/boardengine/legacy.go`
- `internal/boardengine/task_test.go`
- `internal/boardengine/store_test.go`
- `internal/boardengine/board_test.go`
- `internal/boardengine/render_test.go`
- `internal/boardengine/layer_test.go`
- `internal/boardengine/legacy_test.go`
- `internal/boardengine/boardtest/bench_test.go`
- `internal/boardengine/boardtest/bench_cli_test.go`
- `internal/boardengine/boardtest/concurrency_test.go`
- `internal/boardcli/cli.go`
- `internal/boardcli/text.go`
- `internal/boardcli/text_test.go`
- `internal/boardcli/help_test.go`
- `internal/boardcli/verbs_test.go`
- `internal/boardcli/cli_test.go`
- `internal/loomcli/landingdeps_integration_test.go`
- `internal/battencli/lifecycle_integration_test.go`

**Create:**
- `internal/boardengine/labels.go`
- `internal/boardengine/labels_test.go`

**Uses:**
- `internal/boardengine/config.go`

**Intent:**
Swap the entry model from tier plus type to kind plus labels (D1, D2, D3, D4, the README half of D6 and D7).
Removing `Tier` and `Type` forces every reader of them into this one card; the on-load migration of old `board.json` entries is card 4.

Model (`task.go`):
- `Task` loses `Tier` and `Type` and gains `Kind string` (`json:"kind"`, `task` or `note`) and `Labels []string` (`json:"labels"`).
  Delete `MinTier`, `MaxTier`, `DefaultTier`, `DefaultType`, `entryTypes` and `validateTask`'s tier and type checks;
  declare `KindTask` and `KindNote` constants, and validate that `Kind` is one of them.
  `NewTask` defaults `Kind` to `note` and `Labels` to an empty list.
- `labels` replaces the whole list on upsert (the JSON round-trip already does this).

Label validation (`labels.go`, D3): `validateLabels(t Task, v Vocabulary) error` refuses, each with a message naming the entry's slug, the label where there is one, and `board.yaml`'s list as the way forward:
- a label in neither list (add it to `board.yaml`'s `types` or `labels`, or drop it from the entry);
- a duplicate label;
- a note with zero or more than one type label, naming the type labels it carries;
- a task with no type label.
Order is preserved as given.

Store (`store.go`):
- `Store` gains an unexported `vocab *Vocabulary`.
  `Board` sets it on every store it builds, read and write; it is nil only for a `Store` built directly by `NewStore`, which only store-level tests do, and nil skips `validateLabels` alone.
  Bound: no production write reaches a store without a vocabulary, because every write goes through `Board.boardCriticalSection`.
- `validateWrite` keeps the dangling, isolated and cycle checks, drops the tier call, and adds D2's kind rules and the label check:
  a note with a non-empty `depends_on` is refused (clear it with `lyx board set-deps`, or make the entry a task with `lyx board promote`);
  a `depends_on` target that is a note, done or not, is refused (promote that entry, or drop the dependency with `set-deps`);
  an incoming note that any other entry depends on is refused, naming the dependents (clear their `depends_on` with `set-deps`, or keep the entry a task).
  Then `validateLabels` runs on the incoming entry when the store has a vocabulary.
- `validateTier` is deleted.
  `SetStatus` runs `validateWrite` on the entry with its new status instead, so a status write is refused while the entry carries an unconfigured label, as D12 requires.
  `Prune` and `RemoveTask` stay unvalidated: they create or change no entry beyond stripping removed slugs from survivors' `depends_on`, which no rule can turn invalid.
- `upsertAllowedKeys` drops `tier` and `type` and adds `kind` and `labels`.
  `validateUpsertFields` keeps naming the way forward: `tier` is retired (use `"kind"`), `type` is retired (use `"labels"`), and `deferred` points at `"kind": "note"`.
- `BriefTask` replaces `Tier` and `Type` with `Kind` and `Labels`.
- `Store.Promote(idOrSlug any)` sets `Kind` to `task` and runs `validateWrite`;
  a task is returned unchanged with a flag telling the caller nothing changed.

Board (`board.go`):
- `Board` keeps the config's `Vocabulary` from `New` and sets it on the store in `boardCriticalSection` and `loadStore`.
- `Board.Promote(idOrSlug any)` returns `noWrite` for an entry that is already a task (D4), so no save, render or sync happens.
- Delete `PromoteNote` (its last caller went in card 2).
- No code gates claimability on tier; "only a task is claimable" lives in the Tasks section meaning in `render.go` and the CLI help, both rewritten in this card.

Order and render (`layer.go`, `render.go`, D7):
- `RenderOrder` orders open tasks first, then open notes, then done entries; within each, by dependency layer, then ID.
- The README is `# Board`, an intro without tier wording, `## Tasks` split into dependency layers as today, `## Notes` with one `###` section per type in `Outputs.Types` order, its heading the type name capitalised with an `s` appended (`Bugs`, `Enhancements`), then `### Other` for a note whose type label is no longer configured, then `## Done`.
  A Notes subsection with no entries is omitted, as an empty layer is today.
- Every README entry line shows its labels, joined with `, `, between the slug and the status: `` `slug` · bug, area · status ``.
- The design-doc header's metadata line names the kind (`Task` or `Note`) where it named the tier section, then the labels and status.
- Delete `tierSections` and `tierName`; the section names and meaning lines for Tasks, Notes and Done stay declared in `render.go` alone.

Legacy (`legacy.go`): `entryFromLegacy` sets `Kind` (a non-deferred task becomes `task`, a deferred task or a note becomes `note`) and `Labels: ["enhancement"]` in place of the retired tier and type.
Card 4 routes this through the general migration.

CLI (`boardcli`):
- `text.go`: `RenderCompact` prints kind, slug, title, the labels joined by commas, then `[status]` when set, each column padded as today (D6).
- `cli.go`: `promote` takes `{"slug"}` or `{"id"}` through `resolveLookup`, so a payload with `tier` is refused as an unknown key (D4); its help says it turns a note into a task and returns a task unchanged, and that demotion goes through `upsert` with `"kind":"note"`.
  The root `Long`, the `upsert` field list (`kind`, `labels` in place of `tier`, `type`), the `upsert`/`upsert-batch` examples, and the `list`/`find` `--text` column descriptions drop every tier and type word.

Tests:
- `labels_test.go` covers every `validateLabels` refusal and a valid note and task, asserting the exact message names the slug, label and `board.yaml`.
- `store_test.go`, `task_test.go`, `board_test.go`: replace every tier and type case with the kind rules (note with deps, dependency on an open and on a done note, demoting a task that has deps and one that is a dependency, each refused with the way forward), label validation through a store with a vocabulary, `SetStatus` refused on an unconfigured label, `Promote` of a note and of a task (no write, asserted by `board.json`'s modification time or content staying the same), and the `tier`/`type` upsert keys refused.
  Configs built as literals in tests gain `Types` and `Labels`.
- `render_test.go`, `layer_test.go`: the new section layout, type order from `Outputs.Types`, the `Other` section, omitted empty sections, labels on every line, and the absence of `Next Up` and `tier`.
- `text_test.go`, `help_test.go`, `verbs_test.go`, `cli_test.go`: the new columns, help text and promote payload.
- Every test upsert that creates an entry through a config-built `Board` now passes a valid `labels` (and `kind` where it needs a task): `boardtest`'s benchmarks and concurrency test, `landingdeps_integration_test.go`, `lifecycle_integration_test.go`.

**ImpactSummary:** Replaces tier and type across boardengine and boardcli; outside them only test fixtures in boardtest, loomcli and battencli change, and no production caller reads tier or type.
