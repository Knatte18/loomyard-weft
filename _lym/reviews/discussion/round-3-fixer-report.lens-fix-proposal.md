```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I tested each proposal the fixer carried out (F1–F6) for a concrete case where the change itself is wrong.

**Findings**

1. **MEDIUM. F4's resolution never ties the run directory to the resolved worktree.**
   - **Changed text:** D3's last paragraph, "the verb resolves the worktree through `lymcwd` alone (`PATTERN-cwd-resolution`), from the seam cwd and never from the file's path".
   - **Case:** a session in worktree A runs `lym burler validate-review` on worktree B's review. The path can be absolute, and the verb accepts one (`internal/burlercli/validate.go:53`).
   - **What goes wrong:** the builder reads B's run directory and round. It takes the decision record path, the anchor path and therefore the parent-review files from A. That yields refusals or passes against the wrong record. Before the fix, resolving from the file's path at least kept the two together.
   - **Evidence:** `_lym` links to the weft sibling (`_lym -&gt; …/review-rounds-converge-weft/_lym`). `lymcwd.ResolveWithin` maps a records sibling back to its owner worktree (`internal/lymcwd/lymcwd.go:135-163`), so the check is available.
   - **Missing rule:** the record needs one sentence: when the file's run directory does not lie under the resolved worktree's `_lym`, the verb runs the shape checks alone. Row D3's acceptance row on `lym burler validate-review` needs the same case.

2. **LOW. F2's class lock has a cost the record does not state.**
   - **Changed text:** "A carried finding keeps its earlier finding's class", plus the matching refusal and acceptance row.
   - **Case 1:** a held `decision`-class finding that a ruling partly settled leaves a remainder that is fixer work. It must stay `decision` and be escalated to the parent again.
   - **Case 2:** a `design` finding that turns out to need the parent's call cannot become `decision`.
   - **Only way out:** mark it `outdated` and raise a new finding, but D4 holds a new finding to BLOCKING severity and a changed file.
   - **Fix:** state the trade-off in D3 or the Auto-mode assumptions, or allow a carried finding to move between `design` and `decision` only.

3. **LOW. F3's accepted bound says the parent wrote every listed entry, which is wider than true.**
   - **Changed text:** "the block lists only entries the parent added through `lym loom decision add` since round N-1 began".
   - **Evidence:** entries can come from the parent or the operator. The prompt says so: "a decision your parent or the operator recorded for this run". D2's digest key lists every added entry, whoever wrote it.
   - **Fix:** write "the parent or the operator". The bound still holds with that wording.

I found no case against F1, F5 or F6:
- **F6:** a digest key cannot misfire on a replace, because `lym loom decision replace` folds every added entry, and `lym loom decision add` only appends.

Tool calls:
- `ls -la _lym` and a grep for writers of `decision add`: to confirm the `_lym` link target for finding 1 and who appends added entries for finding 3. My context showed neither.
- A grep in `internal/discussionparser/carryover.go` and `internal/loomshed/amendment.go`: to check whether any Go writer edits an added entry's text, which would break F6's digest. My context did not show these files.
