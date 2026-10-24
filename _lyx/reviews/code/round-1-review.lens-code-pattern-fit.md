```yaml
lens: code-pattern-fit
outcome: reported
```

**Task:** check the `umbrella-groundwork` diff against PATTERN.md, the `pattern/` background files, the repo's CLAUDE.md rules and the declared PATTERN changes (lens code-pattern-fit).

**Rules checked**

- **`PATTERN-cwd-resolution` (declared change): held.** The `PATTERN.md` line carries the declared text exactly ("the hub's workspace files"). The background file's bullet now covers each beam's `&lt;RepoName&gt;-&lt;beam&gt;.code-workspace` file (`pattern/PATTERN-cwd-resolution.md`).
- **`PATTERN-pair-parent` (declared add): held.** `PATTERN.md:27` carries the declared text exactly, placed with the Fabric and git entries. It has no `(test)` marker and no background link; the declared text has neither, so that matches.
- **`PATTERN-owner-machine`: held.** The background file now says the parent comes only through `hubgeom.ParentOf`, and that adopting a beam moves the beam only.
- **`PATTERN-pair-teardown`: held.** The background file's bookend line is changed to "anchored at the prime", as the decision record's Constraints restate.
- **`PATTERN-cli-cobra`: held.**
  - `lyx beam` is a cobra subtree under the root: `beamcli.Command()` is wired at `cmd/lyx/main.go:182`.
  - The `beamcli` deviation (no engine package of its own) is added to `pattern/PATTERN-cli-cobra.md`.
  - `cmd/lyx/sandbox_coverage_test.go` has an exemption row for `beam` with a stated reason.
- **`PATTERN-cliwire-sole-wiring`: held.** `internal/beamcli/cli.go` has no `cliwire` or `Module` reference, which matches the "Added after Discussion" ruling that `beamcli` is hub-only like `adoptcli` and `battencli`.
- **`PATTERN-shed-recipe-registry`: held on a spot check.** The diff touches `internal/shedrecipe/registry.go`, `entries_describe.go` and `recipe.go`, plus `contracts/recipes/beam-recipe.yaml`. I did not read how the row is registered against the one `Constructor` map.
- **`PATTERN-stencil-ownership`: held on a spot check.** `contracts/stencils/orch/orch-template-role.md` is edited as a file under `contracts`. I did not read its content for the parent-beam filter rule.
- **`PATTERN-refusal-way-forward`: held on the code I read.** Every new refusal I read in `beamcli`, `battencli/refusal.go`, `startguard.go` and `beamfork.go` ends with a way forward. I did not check that each has a reaching test.
- **`PATTERN-fabric-git`: held on the code I read.** The weft reset sits in `internal/fabricengine/destroy.go` (178 lines changed). `beamcli` reaches git only through `fabricengine` calls (`MarkBeam`, `ResetRecordsToParent`, `Add`).
- **CLAUDE.md "Docs land in the same commit": held.**
  - `docs/overview.md` gains the `beam` module row and the "beams sit beside the stack" entry.
  - The orch, cleanup and adopt rows are updated.
  - `internal/beamcli/doc.go` exists, and `doc.go` files change in `fabricengine`, `boardengine`, `hubgeom`, `orchcli`, `tenderengine`, `loomcli` and others.
  - `plugins/ly/skills/board/SKILL.md` names the `project` kind, the `P` handle, `order`, `beam` and `parent_beam`, as the decision record's Docs constraint asks.
- **Commit messages and `Pattern-Departure:` trailers: held.** No commit in the range carries a trailer, and each is the plain `N: slug` message with the attribution line.
- **Beam, never umbrella: broken** (finding 1).

**Findings**

1. **LOW, class consistency.** The word "umbrella" is in names the diff adds, where the decision record's Goal requires "beam throughout, never umbrella, in every name, text and doc this task touches".
   - Location: `internal/boardengine/rename_test.go` (added lines in the diff).
   - Summary: `newRenameBoard`'s doc comment and fixture use a project slug `"umbrella"` with title `"Umbrella"`, and the test reads `"umbrella"` back, with a local variable `umbrella` and messages naming it.
   - Evidence: the diff lines `// old carries … the one member of the project umbrella;`, `map[string]any{"slug": "umbrella", "title": "Umbrella", "kind": boardengine.KindProject …}`, `renamedProject, _, err := b.GetTask("umbrella")`.
   - Fix: rename the fixture to a neutral name such as `grouping`.
   - A grep of the added lines found no other occurrence outside this test.

I found no declared PATTERN change left unlanded.

**Tool calls (3 of 6)**
1. `git diff` of `PATTERN.md` and `pattern/`, a grep for "umbrella" and `Pattern-Departure` in the range, and a grep of `PATTERN.md` for the two declared entries. The manifest did not show whether the declared entries landed.
2. A grep of `internal/beamcli/cli.go` for `cliwire`, the `beamcli` wiring in `cmd/lyx` and its sandbox row, the `docs/overview.md` diff and a `pattern/` listing. The manifest lacked the docs and wiring evidence.
3. A grep of the added lines for other "umbrella" hits, plus the `SKILL.md` diff and stencil stat. The first grep was capped at ten lines.
