---
round: 1
focus:
  - "internal/loomcli/wiring.go: do main's pre-push guards on Commit, Spawn and Push and the beam's DefaultEngine fields and assistant-reviewer segment bounds and settings all survive together, with no field dropped or shadowed?"
  - "Files both sides changed (wiring.go, configreg_test.go, configstrictness_test.go, docs/overview.md, PATTERN.md): does each merged result keep both sides' changes, and does every PATTERN entry main touched or the beam added still read consistently with the code?"
  - "Semantic interaction, not textual conflict: where main's changed code (loomcli reviewstatus.go and checklist.go, shedverbs status.go, shedrun childrecipe.go and paths.go, battencli, dobbyengine, beamcli create) reads, calls or wraps code the beam changed (engine blocks, codexengine, Burler assistant seat, shuttle), does it still compile-correctly and behave correctly against the beam's new signatures, defaults and paths?"
  - "No new behaviour: does any hunk in the range go beyond reconciling the two sides, such as an edit to main's logic or the beam's logic that neither side's commits carry?"
  - "Decision record's Bounds name the P2 Sketch as the authority on what the beam's code must keep doing: does the merged tree keep every Codex-engine behaviour the beam added, with nothing of it dropped to make main's code fit?"
  - "Summary claims at _lym/landing/summary.md: check 'Git merged every file without a conflict', the four both-sides files, and 'none of the beam's code that main's commits read changed under it' against the diff."
  - "Test economy and coverage in merge-touched tests: did a merge-time edit to a test duplicate existing coverage, or leave a behaviour the merge itself changed without a test?"
---

The range is the merge's net import of main's nine commits, so main's own logic is already reviewed on main.
The subject is the seams: the files both sides changed and the call sites where main's code meets the beam's changed code.
