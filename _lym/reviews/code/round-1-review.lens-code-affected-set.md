```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change, I list what else reads, asserts or documents it and whether the diff covers it.

**Per-change list**

1. **#58, motif's `profile_recipe` removed.**
   - Set found: `shedbuild.Recipe.ProfileRecipe`, `loomrecipe.ProfileRecipe`, the `prepareRunSettings` caller, `motif-recipe.yaml`, `shed-recipe-spec.md`, and the `shedbuild`, `loomrecipe` and `runprofile` `doc.go` files.
   - A repo-wide grep for `ProfileRecipe` and `profile_recipe` leaves hits only under `docs/research`, which are historical measurement reports.
   - The `template.yaml` header, `defaults.motif` and the four motif blocks are in. The reconcile and lacking tests (`migrate_test`, `configsync_test`, `configcli_integration_test`, `boardcli/verbs_test`, `runprofile_test`) were updated.
   - Covered. No finding.
   - Not verified: the `lym config profiles` help text and the board skill's profile paragraph (`plugins/ly/skills/board/SKILL.md`). Neither was enumerated in my context. Greps of `internal`, `contracts`, `plugins` and `docs/overview.md` for lists of the recipe names (`tapestry, darn, pawl`) found no stale enumeration that omits motif.

2. **#53, `discussed` status.**
   - Set found: `layer.go`, `render.go`, `decision.go`, `doc.go`, `loomcli/wiring.go`, the `text.go` `[status]` listing and the board skill line.
   - `boardcli/text.go:54` prints the raw status, so `[discussed]` needs no change. The `list` layer comes from `ComputeLayers`. The README column and section code is generic over layers.
   - Covered, apart from the finding below.

3. **#49, batten board status and the shedengine start write.**
   - Set found: `battencli/commitstatus.go` and its file header, the `cli.go` doc, `boardengine.SetRunStatus`, the `shedengine` doc and `run.go`, `docs/overview.md`, and `loomcli/doc.go`.
   - The loom seam is unchanged. Its `lastBoardStatus` de-dup still works with the start write.
   - Covered. No finding.

**Findings**

1. **MEDIUM, design.** `internal/battencli/startguard.go:96,117,170`.
   - The start guard refuses a hand-started `lym batten run` or `step` only when the board status equals `StatusDraft`. Its message is "is a draft task, which no run starts; way forward: release it".
   - Before the diff, a finished motif was left at `draft`, so a hand re-run was refused with the release command.
   - A finished motif now carries `discussed`. The guard passes it, so `lym batten run <slug>` starts an already-discussed entry with no release.
   - This contradicts the new text the diff adds: the README's "nothing starts it until its status is cleared" (`render.go` `layerSection`), `decision.go:238-240`, `doc.go:283` ("released the same way") and the board skill.
   - The diff did not touch `startguard.go`, `cancelcmd.go` or `adoptcli`.
   - Fix: the Draft probe should also accept `StatusDiscussed`, with wording naming it, or the claim should be narrowed.

2. **LOW, consistency.** `internal/adoptcli/runreport.go:223`.
   - The `closedStatus` comment gives "such as the draft a cancel writes" and now also covers `discussed`. The behavior is correct, since `discussed` is non-empty, non-run and not done. This is only a doc-currency note and could be left as is.

3. **LOW, consistency.** `internal/boardengine/doc.go:244` and the board `cli.go` `set-status` help (`:640,646,879,881`).
   - These describe "null releases a draft task" and the "not run statuses" list as "draft", "done", "abandoned" and a hand-set word.
   - They do not name `discussed`. `doc.go:283` does mention it, and a null status also releases a discussed task.
   - Mild incompleteness: the help text for `set-status` is not updated.

**Rule documents and commits**

- `PATTERN.md` and the declared-changes list: the constraints section lists no declared changes, and none were expected. No `PATTERN` entry is touched by the diff.
- Entries that bear on the changes:
  - `PATTERN-shed-recipe-registry`: not affected.
  - `PATTERN-no-hard-coded-choice`: `defaults.motif` is a table row.
  - `PATTERN-named-constants`: `StatusDiscussed` is declared once and used in `wiring.go`, `layer.go` and `render.go`. The `"discussed"` literals in tests are fine. The `"draft"` literal in `startguard.go` is already a constant.
- Commit messages carry no `Pattern-Departure:` trailer, and none is needed.
- Stencils: none changed by the diff, and no stencil text found says a motif returns to draft.

**Tool calls**
1. `grep` over `internal`, `contracts`, `plugins` and `docs/overview.md` for the recipe-name enumerations, plus `grep draft` in `boardengine/doc.go` and `boardcli/cli.go`. My context lacked any docs that list the recipe names, or the `set-status` help text.
