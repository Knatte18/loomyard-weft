```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the committed diff (stencil prose only, no Go code).

**List: each branch, the case I ran, and the verdict**

- **`darn-fragment-make-change.md` step 6 (no cold fixer).** A three-step change. Step 1 edits pkg A, then `lym gate test` runs after the last edit of that commit (step 3, line 13). The gate passes, so the writer commits per step 6, line 26. The same cycle repeats for steps 2 and 3. **Correct.** Step 1's "Each commit is one change, with its own reads, edits and gate run" (line 9) agrees with "as soon as its gate passes".
- **Same fragment, a gate that keeps failing on step 2.** The green rule ("each leave the tree green") keeps step 2 uncommitted while step 1 is already committed. "Never hold the change uncommitted until the end" still holds: a stall then loses only step 2. **Correct, no contradiction.**
- **Same fragment, gate answers `done: false` (exit 75).** Step 3 says re-run. The commit waits for a pass. **Correct.**
- **Same fragment, delegated per-package subagents (step 2, line 12).** The writer keeps "the commits" in its own session, so it commits once the subagents return and the gate passes. **Correct.**
- **Same fragment, docs and PATTERN (steps 4–5).** "In the same commits as the code" fits a coherent step that carries its own doc edit. **Correct.**
- **`darn-fragment-make-change-gated.md` step 6 (cold fixer).** A three-step change with no gate run by the writer (step 3, line 24). The writer commits as soon as each step's edits are made (line 46). The verify gate then tests the committed tree after the turn (line 40). **Correct.** The fragment has no per-commit green requirement, and it had none before the change either. Step 1's "Each commit is one change, with its own reads and edits" (line 20) agrees.
- **Gated, reported-test-failure branch (step 1, lines 10–17).** The writer asks the parent and ends its turn before any edit, so there is nothing to commit. **Correct.**
- **Gated, measuring-run exception (step 3).** A one-class run inside a step does not delay that step's commit beyond its edits. **Correct.**
- **Host stop paths (`darn-template-write.md`, lines 57–61, "end your turn with no commit").** Both stops fire in Step 3, before Step 4's edits, so the new commit rule never meets them. **Correct.**
- **Host bug-entry waits ("No test seam" / "No bug fixed", lines 115–118).** "No bug fixed" is asked before code. "No test seam" can come mid-change, and then the steps already finished are committed per step 6. **Correct.**
- **Host re-prompt after handoff (lines 131–134, "fix them with new commits").** This agrees with per-step commits. **Correct.**
- **`sampler-template-agent.md`, the Working bullet (lines 34–35).** It agrees with "Commit what the trial settled as an ordinary change" (line 47) and with the ending's "Commit your work so the worktree is clean" (line 56). **Correct.**
- **Sampler bullet continuation line (line 35).** It is indented two spaces under the list item, so markdown renders it as part of the bullet. **Correct.**

**Findings:** none. The diff changes no function or error path, and every instruction branch I ran through the changed lines is consistent with the steps around it.

Tool calls: none made.
