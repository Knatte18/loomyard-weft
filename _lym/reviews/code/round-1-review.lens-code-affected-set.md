```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set** (what the diff forces outside the lines it touches).

**Per change, the set found:**

1. **Edit-directive text** (`contracts/stencils/edit/edit-directive.md:17-19`)
   - **Renderer:** `internal/editdirective` reads the stencil at call time through `stencilstore`. The new lines are rendered with no code change. Covered.
   - **Engine copies:** `contracts/stencils/edit/` holds only `edit-directive.md`. The Claude and Codex copies are separate engine blocks and tell no seat to write scratch files. The summary's claim that they are unchanged is consistent. Covered.
   - **Edit-rule restatement test:** `TestStencils_NoEditRuleRestatement` matches the phrases `never by shell`, `heredoc` and `edit or write` (`contracts/stencils/stencils.go:910-914`). It exempts the `edit-directive` stencil, and the new sentences contain none of those phrases. Covered.
   - **`internal/editdirective/editdirective_test.go`:** it asserts only the `Edit or Write` anchor and the absence of markers, banner and trailing newline. The new text breaks none of these. Covered.
   - **Other `.scratch` mentions in `contracts/`:**
     - `loom-plan-spec.md:159` says a `Create` target under `.scratch/` is a transient draft.
     - `beam-template-board-tidy.md:27` says to draft a body in `.scratch/`.
     - `orch-role.md:219` and `orch-role.md:441-456` use `.scratch/` for drafts.
     - None of these say `.scratch` is a real directory or that it holds relative-reference builds. The orch role is exempt from the edit directive under the stencil-ownership PATTERN. None is left wrong. Covered.
   - **Fasit's second option (tell cards not to build projects with relative references under it):** the directive tells the agent, not the cards.
     - A card author reading `loom-plan-spec.md:159` is not warned.
     - The directive reaches every spawned role, so the agent that runs the card sees the warning. I am not filing this.

2. **`internal/editdirective/doc.go`** (lines 16-18)
   - It mirrors the directive. It says "kept for the task alone" and "a build there resolves ... from the link's target".
   - Neither it nor the directive names the weft worktree, which is consistent with commit `9fe6ce64` (fabric vocabulary kept out of the directive). The doc comment keeps that vocabulary out of `editdirective` too. Covered.

3. **`internal/fabricengine/doc.go`** (lines 654-657)
   - It states the target is in the weft worktree and that the build does not resolve from the warp anchor. It matches `scratchLinkTarget` (`<weft worktree>/<anchor>/.lym/scratch`, doc.go:649). Covered.
   - It names "the edit directive" from fabric, which the fabric boundary does not forbid. A fabric doc comment that names the edit directive is not a violation of the rule that fabric learns nothing of IDEs, shuttle, the board or heddle. I did not read `pattern/PATTERN-fabric-boundary.md`.
   - `internal/fabricengine/scratchlink.go:2-3` says the link's modification time never changes. It says nothing contradicting the new cost. Covered.

4. **Declared PATTERN changes:** the rubric's amendment and constraints name none, and the diff touches no `PATTERN.md` or `pattern/` file. `pattern/PATTERN-stencil-ownership.md` says the `edit-directive` stencil is the only statement of the edit rules, and the change keeps that. Nothing to raise.

5. **Commit messages:** no `Pattern-Departure:` trailer, and none is needed. The first commit's reason (go test cache hits) matches `fabricengine/doc.go:651-653`.

**Findings:** none.

**Not found:** the repo has no `docs/` page or golden file asserting the directive's text. A grep for `Scratch files go under` matched only the stencil and `editdirective/doc.go`.

**Tool calls (3):**
1. `grep` for `editRulePhrases` and weft in the stencils. I needed the phrase list, which the manifest lacked, and the first grep returned nothing.
2. `grep` for `editRulePhrases` across the repo, to find where it is defined.
3. `Read` of `contracts/stencils/stencils.go:905-924`, for the phrase list.
