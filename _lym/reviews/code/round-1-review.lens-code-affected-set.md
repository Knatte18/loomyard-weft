```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, covering what the diff forces outside its own lines.

**Per change, the set found**

1. **Sweep paragraph in the implementer stencil, `contracts/stencils/webster/webster-body-implementer.md:55-58`.**
   - Other stencils and code that restate step 2: `webster-prefix-fork.md` and `webster-prefix-recovery.md` have no step-2 text. The grep for `quarry resolve`, `sanctioned` and `grep` found nothing in them.
   - `internal/websterengine/recordbatch.go:125` only mentions `lym quarry resolve` in a refusal message. It is unrelated, so no change is needed. Covered.
   - `tools/sandbox/SANDBOX-WEBSTER-SUITE.md:49` uses "sanctioned deviation" for a different thing, a sandbox black-box rule. No change is needed. Covered.
   - `PATTERN.md` has no entry about the sweep, and the focus directive declares no PATTERN change. Covered.

2. **"The second sanctioned deviation" at `:58`, against the bound at `:43` and `:69`.**
   - `:43` defines the second deviation as "a fix to a failure the card caused, under the bounded self-fix section's two-pronged test and its attempt cap". `:69` calls the guard-step fix "the first sanctioned deviation in step 2's rule".
   - The ordering stays consistent: guard-step fix first, caused-failure fix second.
   - The new sentence asserts that every hit outside the targets is a failure the card caused. The sweep runs before the first edit and before any gate failure, so the caused-failure test (`:119-122`) does not apply to it. The sentence points at the second deviation without carrying its bound, which reads as unbounded licence to edit any grep hit outside the targets. It is also a pre-gate edit, not a "fix to a failure". Not covered; see F1.

3. **"After any rename tool run" at `:55`, against the Rename paragraph at `:49`.**
   - `:49` already runs the rename tool, then greps for zero references to the old name, then updates the doc-comment opening.
   - The new sweep also runs after the rename tool, so the two greps overlap for symbol renames. The new text does not say that the `:49` grep can be the sweep, nor that the sweep then reads and edits the remaining hits. The ordering is compatible; the wording is redundant but not contradictory. See F2.

4. **Grep instruction at `:56` against `instead of grepping` at `:53`.**
   - `:53` is scoped to locating the card's own glyphs. `:56` is scoped to retired names, and the two are distinct. The sentences are adjacent, so a fork could still read a conflict, but both scopes are explicit. Covered, no finding.

5. **Test `internal/websterengine/template_test.go:801-811`.**
   - The two added `contains` strings match the stencil text at `:56-57` exactly.
   - They assert static stencil prose, so they sit on the test-economy rubric line "a new assertion on a stencil's static prose". That line belongs to the code-tests lens, so I only note it here.
   - Other tests asserting step-2 text, such as `govalues_test.go` and `stencils.go`, were not checked in depth. The grep for `instead of grepping` and `sweep` found no other assertion on this paragraph. Covered.

6. **Commit message.**
   - The message is stencil-only and has no `Pattern-Departure:` trailer, which none was needed. Its description matches the diff.
   - It says "before its first hand edit" and that the fork reads and edits "every hit in one message each". This matches the stencil.

**Findings**

- **F1, MEDIUM, class consistency.** Location: `contracts/stencils/webster/webster-body-implementer.md:58`, against `:43` and `:119-122`.
  - Summary: the sentence labels any sweep hit outside the card's targets a "fix to a failure the card caused", the second sanctioned deviation. The sweep runs before the first edit and before any failure, so the two-pronged test and attempt cap at `:43` and `:119-122` cannot bound it.
  - Suggested fix: reword to say the sweep edit is sanctioned only for a hit that references a name the card retires, and keep the deviation count and its bound at `:43` in step. Alternatively, extend `:43` to name the sweep edit as its own bounded case.

- **F2, LOW, class consistency.** Location: `contracts/stencils/webster/webster-body-implementer.md:55`, against `:49`.
  - Summary: "after any rename tool run" duplicates the grep already required at `:49` for symbol renames. The text does not say whether the `:49` grep satisfies the sweep's grep.
  - Suggested fix: add that the `:49` grep may serve as the sweep's single Bash grep when it covers the whole worktree. Otherwise state that the sweep replaces it.

**Tool calls**
1. A grep across `contracts/`, `internal/websterengine`, the webster sandbox suite and `PATTERN.md` for `sweep`, `sanctioned` and `instead of grepping`. My context did not hold other stencils or tests restating step 2.
