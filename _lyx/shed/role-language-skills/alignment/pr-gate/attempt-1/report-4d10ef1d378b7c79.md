---
verdict: APPROVED
---

The diff of pull request 641 at `1f003707c` against `main` matches the board entry's goal and every decision in the decision record, with no change outside them.

## Against the goal and the decisions

- D1, D2, D3: `internal/languageconfig` registers `language` as `HubWide` in alphabetical order in `configreg.Modules()`, with an empty `skills: []` template default and `skills` classified Guard.
  `Load` returns the empty list for an empty board dir and goes through `configengine.LoadOrTemplate`, so an absent hub file yields the empty list.
  Content failures name `skills` and end with the `lyx config language` way forward; `contracts/specs/refusal-spec.md` gains a loom `language.yaml invalid` row and extends the webster invalid and unreadable rows.
  `pattern/PATTERN-config-strictness.md` lists `languageconfig` among the degrading callers.
- D4: `AppendSkills` keeps the role's own list first and drops repeats.
  The darn writer, the Webster master (`RunDeps.LanguageSkills`), the recovery strand (`RecoverDeps.LanguageSkills`, loaded in `lyx webster`'s own `wireHub`) and the conflict session at Merge-In and Finalize (through `landingshed.Deps` and `mergeresolve.Deps`, loaded in `loomcli`'s `wire`) all take it.
  The role lists stay package variables.
- D5: `burlerengine.Profile.LanguageSkills` selects `seatSkills`.
  `burlerRoundProfile`, `burlerFixProfile` and `lyx burler run`'s `decodeProfile` accept `language-skills` and refuse a non-boolean.
  Only the Webster, Merge and Darn review and fix rows set it, pinned by `TestShippedRecipes_LanguageSkillsOnCodeRounds`.
- D6: `darnSkills` gains `scribe:testing`.
- D7: the template comment steers toward comment and testing skills and away from build skills; no code filters a skill name.
- Out of scope held: no other role's list changes, the hub's own `language.yaml` value is not set, and no language detection is added.
  The `wireLanding` extraction in `internal/loomcli` serves D4's route from `arm.go` and `sharedbootstrap.go` to `landingDeps`.

## Carried-over risks

- Plan-Review round 2's open `card6-unreadable-row-and-master-stop-doc-stale` is resolved in the diff: the refusal-spec `config file unreadable` row and the "A broken config stops Master" section of `internal/websterengine/doc.go` both name `language.yaml`.
- Webster-Review round 1's F1 (stale `buildLoomShed` doc comment) is fixed by `0796c0115` and `0ed811235`.
- The decision record's open risks need operator action after landing, chiefly setting this hub's `language.yaml`, and none is a scope gap.
  `burler-fixer-fork` is left to decide whether the fix rows keep `language-skills`, as the record says.

## Overlap and landing order

- Peer `kickstart-per-card` (row 7, Plan-Write; this run's `own_row` is 22): it lists no changed paths yet.
  Its design changes the Webster fork prompt's kick-start pack layout and adds `lyx loom card-pack`, while this run changes role skill lists and config loading.
  Neither changes the other's behaviour or a shared PATTERN entry.
  Likely text-only overlap at different places: `internal/websterengine/doc.go`, `contracts/specs/refusal-spec.md` and `internal/loomcli`.
  Landing order: this run (row 22) lands first, and `kickstart-per-card` takes these changes at Merge-In.
- Parent commits: `lyx loom peers` lists none.
  `main`'s `tools/forkaudit` commit `5e7adbbe2` is already merged into this branch and touches none of this run's behaviour.
- Shared bookkeeping files: `cmd/lyx/testdata/test-budget.yaml` and the `docs/overview.md` module-table line for `internal/languageconfig`.

## Superseded

None: `engine-model-choice` depends on this task and stays open, and `burler-fixer-fork` and `dotnet-scoped-verify` are only referenced.
