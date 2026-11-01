```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens review of the machine-switch plan, cards 1–14.

**Findings**

1. **BLOCKING** (Card 7, D9): the shedrecipe Darn-Write entry and its test env are missing.
   - `internal/shedrecipe#darnWriteEntry` checks each Darn path for an absolute root against a fixed list (`internal/shedrecipe/entries_darnwrite.go:38-41`). That list names `Darn.StopMarkerPath` but would not name the new `ConditionStopMarkerPath`.
   - `internal/shedrecipe#darnTestEnv` builds `loomshed.DarnDeps` by hand (`entries_darnwrite_test.go:44`) and leaves the new field empty. `TestDarnWriteEntry_HappyPath` runs the producer's `Call` (`entries_darnwrite_test.go:84-100`). Card 7 makes `darnWrite.checkSeams` refuse an unwired `ConditionStopMarkerPath`, so that test fails.
   - Card 7 lists `envkit#FullEnv` and the loomshed fixture but neither of these two members. `TestDarnWriteEntry_ConstructionFailures` (`:53-82`) would also need a relative-path row for the new field.
2. **MEDIUM** (Card 10, D5, help text follows): `beam status` help is missed. `internal/beamcli#statusCmd`'s long help describes "The adopt_check section" (`internal/beamcli/cli.go:78`, inside `statusCmd` 64-93). Card 10 renames the field to `adopt_checks` with one item per subject, but lists only `statusResult` and `dobbyStatus`.
3. **MEDIUM** (Cards 10, 11 and 12, D1/D4/D5): `docs/overview.md` is not listed by any card, and it goes stale.
   - `:446` says adopt "lifts the owner pauses and resumes each moved run as a detached first pass". Card 11 removes the resumes.
   - `:447` says adopt "starts the adopt check once, detached, in the prime". Card 10 makes it one check per subject, in that subject's worktree.
   - `:457` describes one "adopt check section" in `lym beam status`.
   - The darn stencil's own doc rule makes the repository overview doc part of the change.
4. **LOW** (Card 10): `pattern/PATTERN-test-speed.md:37` describes the `adopt` gate as `lym adopt check`, run in-process. The verb becomes `lym adopt check <subject>` in the subject's worktree, and no card updates the row.
5. **LOW** (Card 3, D8): `internal/boardengine#replayFields` lists every entry field a replay merges, and its doc names the fields it leaves out (`internal/boardengine/replay.go:143-145`, with `notify_on_land` registered at `:164`). The new `Front` and `OnLand` fields are neither registered there nor named in that comment.
6. **LOW** (Card 3): `internal/boardengine/runlock.go:185-186` lists which `Task` fields a run-held comparison skips (`RunSettings`, `Profile`, `NotifyOnLand`, `OwnerMachine`). It is silent on `front` and `on_land`, and no card states whether they belong in the compare.
7. **LOW** (Card 9): the package of `TestTaskBranchSlugs` likely mismatches. The handle `plan:internal/fabricengine#TestTaskBranchSlugs` is the internal package, but the Intent places it with "the package's other git tests". Those tier1 git tests are `package fabricengine_test`, for example `add_anchor_integration_test.go`. The handle unit should be `internal/fabricengine_test`, or the Intent should name an internal-package file.
8. **LOW** (Card 7): the new wiring goes unasserted. `internal/loomcli#TestWire_DarnRecipe` pins each Darn path in a table (`internal/loomcli/wiring_darn_test.go:128`). Card 7 wires `ConditionStopMarkerPath` in `darnDeps` and `wireDarn` but lists that test only under Uses, so nothing asserts the new path.
9. **LOW** (Cards 5 and 10, order): from Card 5 on, `fronttask`'s action spawns `lym adopt check <subject>`, and Card 6 wires it into dobby. The verb takes no arguments until Card 10, so between cards 6 and 9 an `adopt-check` action would fail on cobra's argument check. Nothing files a front task before Card 13, so nothing reaches it, but the dependency on a later card is unstated.

**Tool calls**
- Searched the repository for `adopt_check`, `suite_warning`, `ReadCheckView`, the `adopt check:` notice text, the stop-marker path name, `task_headroom`, and battencli and boardengine imports of beamcli, adoptcli or fronttask. My context lacked the readers outside adoptcli and the import graph.
- Read the board's field lists (replay, runlock, task validation, the upsert `Use` token), the members around the beamcli, shedrecipe and loomcli lines, the fabricengine test package clauses, and the arbiterstop signatures. My context lacked the enclosing members and those registration tables.
- Read `replayFields`, the boardcli `upsertPayload`, `darnWriteEntry`'s path list and `TestDarnWriteEntry_HappyPath`, and the hibernate wake's caller handling. My context lacked those bodies.
- Searched markdown docs mentioning `lym adopt`. My context lacked the repository docs and pattern background files.
- Read the adopt lines in `PATTERN-owner-machine.md`, the orch role stencil and the board skill. My context lacked them, and they named no step the cards change.
