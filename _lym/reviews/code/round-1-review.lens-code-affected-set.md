```yaml
lens: code-affected-set
outcome: reported
```

Task: work the code-affected-set lens for this diff, which changes only the wording of three stencils.

**Affected set, per change:**

1. **`darn-fragment-make-change.md` step 6** (commit after each coherent step, once its gate passes, never hold the change uncommitted). Covered.
   - Steps 1 and 3 of the same fragment still match the new line ("Each commit is one change, with its own reads, edits and gate run"; "Test with `lym gate test` after the last edit of each commit").
   - Step 2 still keeps "the commits" in the writer's own session.
   - In the host `darn-template-write.md`, Step 3's stop paths still end "with no commit", and "What happens after you hand off" still uses new commits for fixes.
   - The old "each leaving the tree green" rule is kept as "each leave the tree green".
   - The Codex engine renders the same fragment: there is no engine-specific copy.
   - No Go test or fixture asserts the line. The only test reference is a stencil name in an error struct at `internal/loomcli/wiring_darn_test.go:254`, not the text.
   - The `docs/research/.../prompt*.md` copies of the old wording are captured bench data, which the prose rule exempts as measurement reports.

2. **`darn-fragment-make-change-gated.md` step 6** (commit after each step's edits are made). Covered.
   - Step 1 ("Each commit is one change, with its own reads and edits") and step 3 ("the row's verify gate tests your committed tree") still match it.
   - The writer runs no gate in this variant, so leaving out any wait for a gate before committing is consistent.
   - The old rule "small, coherent" survives as "coherent step … small commits".
   - No test asserts the line.

3. **`sampler-template-agent.md` Working bullet**. Covered.
   - It still matches "Commit what the trial settled as an ordinary change" in the trial section.
   - It still matches "Commit your work so the worktree is clean" in the operator's ending.
   - No test asserts the line.

**Rubric checks:**
- Stencil wording: no rule is dropped, weakened or contradicted.
- Repository rules: `PATTERN-stencil-ownership` holds, because the wording sits in stencils read at call time and is not tied to one engine. `PATTERN-refusal-way-forward` holds, because committing is not a step the agents' settings deny.
- Declared PATTERN changes: the directive declares none, so there is nothing to land.
- Commit messages and departures: the one commit (`56763d2e9`) describes the diff accurately, and it carries no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, none of which this change needs.

**Findings:** none.

Tool calls:
- grep across `internal`, `docs` and `contracts` for the changed wording and for test references to the two fragment and sampler stencil names, to find tests or docs that assert the lines; the manifest's grep covered only the old wording.
