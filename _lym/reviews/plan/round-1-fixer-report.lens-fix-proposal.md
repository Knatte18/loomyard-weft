```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer carried out (F1's moved targets, F2–F7) and found one that is wrong.

**MEDIUM — card 3, the trimmed `TestSessionSpec` sentence (F3, fixed as proposed).**
- **Changed text:** `03-sampler-stencil.md` Intent, the `TestSessionSpec` sentence. It now ends at "…the edit directive, the record path and the slug."
- **Summary:** cutting the static-prose assertions leaves the D8 and D9 acceptance criteria with no planned check. The reviewer's claim that "the stencil-wide scans the card names cover it" is false for every clause except ask-the-operator.
- **Evidence:**
  - The acceptance criteria in `_lym/discussion/decision-record.md`, from line 199 on, say: "D8: the rendered stencil … names `sampler-done`, `decision add --by operator` and notes-only board writes". They also say: "D9: the stencil names the no-dev-build-in-this-hub rule and the own `tmux -L` server rule for a trial, and names no denied form."
  - None of the scans card 3 names (`TestStencils_OpeningStencilsRenderParentDirective`, `…RenderEditDirective`, `TestStencils_NoUnboundedAskOperator`, `TestStencils_NoSkillLoadInstructions`, `TestRegistry_MatchesOnDiskTree`) checks that a stencil names `sampler-done`, `decision add`, `lym board` or `tmux -L`.
  - `grep "git push\|gh pr\|deniedForm\|DeniedStep" contracts/stencils/*_test.go` finds nothing, so no stencil-wide scan covers D9's "names no denied form" either.
  - After the fix, card 3 carries D8 and D9 only as stencil prose with no acceptance tag, and no other card claims them. The earlier plan-review history treats a criterion that no card carries as a MEDIUM coverage finding (`_lym/plan/lens-review.md:6`).
- **Possible fixes:**
  - Keep a one-line acceptance statement in card 3 saying D8 and D9 are met by the stencil text the card writes, verified by reading it at review, and say why no test pins it (the rubric's test-economy rule).
  - Or keep only the denied-form negative assertion (`git push`, `gh pr`), since no scan covers it.

**No finding on the others:**
- **F1:** card 5's yaml is only parsed by `RecipeEngines`. `embeddedRecipeSource` stays in card 6, so nothing builds the recipe against the registry or the settings at card 5. `contracts/recipes` has no test listing the directory.
- **F2:** the change to `Deps` is doc-only.
- **F4:** the file target covers both line 1 and `EscalateHalt`.
- **F5:** the template comment is not pinned by any test. The `step_idle_timeout_min` tests in `internal/loomengine/config_test.go` read values, not comments.
- **F6:** the edits to the `Rows` and `RowNames` docs are consistent with each other.
- **F7:** the ImpactSummary now matches the Intent's "`buildLoomShed` … is not edited".

Calls I made, each for something my context lacked:
1. Listed `contracts/recipes` and grepped its tests, the `step_idle_timeout_min` tests and the cards' D8/D9 references: to see whether card 5's new yaml or card 12's template comment is pinned anywhere.
2. Grepped `internal/loomengine/config_test.go` and listed `_lym`: to see whether the template comment is pinned and where the decision record lives.
3. Listed `_lym/discussion` and grepped the decision record for D8 and D9: their text was not in my context.
4. Read `decision-record.md` lines 135–170 and found the acceptance section: the full D8 and D9 text and where the acceptance criteria start.
5. Grepped the acceptance criteria for D8 and D9: the criteria F3's cut affects.
6. Grepped the stencil tests for denied-form scans: to test the reviewer's claim that the existing scans cover D9.
