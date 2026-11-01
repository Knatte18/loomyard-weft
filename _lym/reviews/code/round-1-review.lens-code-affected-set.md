```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens for the diff `abcbaddf..HEAD`.

**Per-change list**

1. New entry `lym-plan-edits-before-gate` in `internal/heddleengine/shipped.json`
   - **Set found:** `shipped_test.go` (shipped-layer validity and duplicate checks), `render_test.go` and `override_test.go` (use `shippedStore.Ordered()[0]`, which is still `lym-batching`), `heddlecli/cli_integration_test.go:159` (iterates `Layers.Ordered()`), `heddlecli/verbs.go` help examples (name `lym-batching` only), `heddleengine/doc.go`, `PATTERN-heddle-instructions` and its background file (none pins entry count or names), and a repo-wide grep for fixtures or goldens of the rendered text (none).
   - **Covered:** yes. No consumer pins the entry count, so none breaks. The pin in `store_test.go` was updated.
   - **Declared PATTERN changes:** the rubric's directive declares none, and no PATTERN edit is needed. The entry sits in the shipped layer, which `PATTERN-lym-instructions-in-code` licenses.

2. Batching lines removed from stencils
   - **Set found:** a grep of `contracts/stencils` and the rest of the repo for each removed sentence found no test or fixture asserting any of them. The only remaining hits are `docs/research/*` reports, which are measurement records and exempt.
   - **Covered:** yes for the removed lines. A batching line still remains in one touched stencil, as finding F1 below.

3. `darn-template-write.md` Step 4
   - **Set found:** the old rule said "once, after the last edit of each commit, never after each edit". The new text keeps "after the last edit of each commit" and "Each commit is one change". The rule "never after each edit" is gone from the stencil; the shipped entry now carries it as "Never run the gate between the edits of one change".
   - **Covered:** yes. The card-specific unit (a commit) is kept.

4. `webster-body-implementer.md`
   - **Set found:** line 51 now reads "The card is one change: the spans it names are its reads, and its gate… runs after its last edit". Line 58 still reads "Read every hit's span in one message, then edit every hit in one message".
   - **Covered:** line 58 is not covered (F1).

5. `burler-step-1-explore.md` and `{{.tool_use_rules}}`
   - **Set found:** the stencil's sentence "Make the reads your focus needs… in one message" became "Read what your focus needs…". The review prompt above still carries the rendered `{{.tool_use_rules}}` text, "Make the reads your focus needs, or the whole target when no focus is set, in one message." That text comes from a separate source: the section under `## Tool-use rules` in the instruction-1 file I read, which is outside the diff. I did not locate the source of that rendered line.
   - **Covered:** unverified (F2).

6. Untouched stencils with batching wording
   - **Set found:** `edit/edit-directive.md:8-10` (one file's edits in one message; a rename across files as one Edit per file in one message), `webster-body-implementer.md:58`, and fork-spawn lines in `burler-fix-self-check.md:10`, `twill-directive.md:13` and `webster-template-master.md:126`.
   - **Covered:** the fork-spawn lines are not read/edit batching and are out of scope. `edit-directive.md` is the `edit-with-edit-tool` rule, which the record keeps, so it is not a finding. That leaves line 58 as F1.

**Findings**

- **F1, MEDIUM, scope.** `contracts/stencils/webster/webster-body-implementer.md:58` still says "Read every hit's span in one message, then edit every hit in one message, so no stale reference is left for the gate to find one failure at a time." This is a `turn-batching` read-and-edit batching line in a stencil the diff touched. The record says the new entry is the one home of the rule and the same change removes the stencils' batching lines. It keeps card-specific parts, and this one is arguably the retired-name sweep. Either remove the batching wording and keep only the grep-the-whole-worktree instruction, or the handler judges it card-specific and drops the finding. Evidence: `git diff` for that file touches only line 51 and leaves line 58 in context.

- **F2, LOW, consistency.** The rendered `{{.tool_use_rules}}` / burler explore text in the review prompt still carries the batching line "Make the reads your focus needs… in one message" (instruction-1 file, under Tool-use rules). The diff removed the same sentence from `burler-step-1-explore.md`, so the stencil and the prompt's rendered tool-use block disagree. If `{{.tool_use_rules}}` expands from Go code or a stencil outside the diff, the rule has a second home and the sentence should go there as well. I did not check the source, so the handler should confirm it before keeping this finding.

**Tool calls**
1. Grep over the repo for the removed batching sentences, `lym-batching` and `Request every call`, to find leftover wording and test or fixture assertions that the diff context lacked.
2. Grep of `*_test.go` for other consumers of the shipped layer or `claude-file`, plus `doc.go` lines 1-20, to check for tests or docs pinning the entry count.
