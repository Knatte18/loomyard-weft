```yaml
lens: code-affected-set
outcome: reported
```

Task: list what the diff's changes force outside the lines it touches, per change, and report the ones left wrong.

**Per change, the set found**

1. **Closed recipe lists for `hardener`.**
   - Covered: `shedcli` table, `shedrun.RecipeNames`, batten `childRecipeSupported` and its refusal text, and the `loomrecipe` embedded-recipe, Finalize-row, Bouncer-row and engine-set lookups.
   - Covered: `reportengine` `runRecipes` and the timing phase table, `boardcli` recipe help, and the `boardengine` task-recipe error text.
   - Covered: `loomcli` hard cap, via the `perchHardCap` special case for `Harden-Bouncer` instead of a `loomPerches` row.
   - Not covered: finding F1 below, which lists prose and a template comment that still name the old recipe set.
2. **Gate keys `harden-live`, `harden-live-copies` and `harden-burler-fix`.**
   - Covered: `gateslot` template, `goReadGates`, the `gateslot` docs table, `PATTERN-test-speed` table and the `PATTERN-gate-slot` Verify list.
   - Not covered: F2, a `PATTERN-gate-slot` line left unchanged (NIT).
3. **`HardenFix` in `LoomBuildEngines`, and the segmented-build rule.**
   - Covered: the `shedbuild` doc and refusals, the way-forward engine list, the `shedrecipe` doc and the conformance, registry and `shedbuild` tests.
4. **Treadle and Tenter removal.**
   - Covered: the stencil registry, the role-opening stencils, the agents template, the roles list and the shuttleparity producers.
   - Covered: `.gitattributes`, the glossary rows, the `docs/overview.md` lines, the `cmd/lym` path rules and the seam-enforcement comments.
   - Covered: the `agents` reinterpretation is registered in `configreg`, with its test row.
   - Covered: a grep finds no live `treadleengine` or `Tenter` name left.
   - The remaining `crucible` hits are the three history kinds card 20 lists as staying.
5. **Crucible deletion.**
   - Covered: `fabricvocab.docRoots`, `PATTERN-fabric-boundary`, the `codestats` comments, and the `overview` link and bullet.
6. **`lym harden` command.**
   - Covered: `cmd/lym` wiring and the module list, the sandbox-coverage exclusion, the tree-sitter allowlist, the `PATTERN-cli-cobra` deviation line, and the strict-caller line in `PATTERN-config-layers`.
7. **`PATTERN-child-processes` change.**
   - Landed as declared, in both `PATTERN.md` and the background file, with the matching `runproc` doc.
8. **`hardener.yaml` config unit.**
   - Covered: `configreg` module with its `CheckValues` and `CheckLoad`, the `HubWide` expectation, the local-key classification test, and the run-settings rows.
   - Covered: the `cmd/lym` strictness set.
9. **Landing, teardown and Bouncer seams.** `landingshed` `Reap` and its doc, `pairteardown` clear and its doc, and the Bouncer `RoundFacts`, `LiveRed` and `OnHalt` seams with docs and tests are all present.

**Findings**

- **F1, LOW, consistency.** Prose and a template comment still name the old recipe set, so a `hardener` task is described as impossible.
  - Evidence: `docs/overview.md:1953` has "tapestry, motif, raddle, arras, darn or sampler" for Run-Shed's child.
  - Evidence: `contracts/orchrole/orch-role.md:44` has "a tapestry, arras, motif, darn or sampler, as the entry's `recipe` says".
  - Evidence: `internal/runprofile/template.yaml:2` lists the recipes that hold a profile block as "tapestry, motif, darn, pawl, sampler, raddle or arras". I did not check whether hardener takes a profile block, so that one is unverified.
  - Fix: add `hardener` to the first two. For the third, add it only if a hardener run reads a profile block.
- **F2, NIT, consistency.** `pattern/PATTERN-gate-slot.md:96` lists the Verify gates ("Code-Burler-Fix, Darn-Write and Drift-Verify gates, Parent-Align and Finalize") without `Harden-Burler-Fix`, although the diff added it at lines ~104. Fix: add the Harden-Burler-Fix `verify` and `live` gates to that sentence.

No finding for "Commit messages and departures": I ran no `git log` call, and the commit subjects in the range carry no visible `Pattern-Departure:` trailer, but I did not read the message bodies. "Declared PATTERN changes" and "Stencil wording" raised nothing.

**Tool calls**
1. `git grep` for the old recipe-set phrases and `code-burler-fix` mentions across docs, contracts and internal: my context did not hold prose outside the diff.
2. `git grep` for `Code-Burler-Fix` and `NameCodeBouncer` in `pattern`, specs and `internal`: to find the per-perch tables and rule docs the diff leaves stale.
3. `sed` of `internal/loomcli/wiring.go` plus greps for `loomPerches` and `RecipeDarn`: to confirm `loomPerches` needs no `Harden-Bouncer` row and to look at `runprofile`.
