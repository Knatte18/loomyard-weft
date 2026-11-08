```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 fix in `internal/fabricengine/unwire_test.go` (new `wantStripped` field and post-`Unwire` exclude assertion) for correctness.

**LOW — `unwire_test.go`, the `tt.wantStripped != ""` block: the assertion has no precondition, so it can pass without testing anything.**
- The check passes whenever `/sub/_lym` is absent after `Unwire`.
- The test never confirms the line was present before `Unwire`. `before[]` captures only `git status`, not the exclude lines.
- If `ReanchorPair` (rest of the function not read, past `reanchor.go:64`) does not re-wire at `sub` and add `/sub/_lym`, the assertion passes vacuously. A wrong `anchorRel` or `names` in `unseedGitExclude` would then still go unseen, which is F1's defect.
- The traced path is otherwise sound:
  - In row `UnwiredPairAloneAtItsAnchor` the unwired pair is at `sub` and the kept pair at `.`.
  - `sameAnchor["_lym"]` is false, so `excludeLinesToStrip` selects `/sub/_lym`. I assumed `excludePatternFor("sub", "_lym")` gives `/sub/_lym`; I did not read it.
  - `containsLine` trims and compares exactly, and `ExcludeLines` reads the repo-common exclude file.
  - The rows with an empty `wantStripped` skip the check, which is correct because their anchored line is shared with the kept pair.
- Fix: before `Unwire`, assert `containsLine(gitkit.ExcludeLines(t, h.PrimeWorktree()), tt.wantStripped)` when `wantStripped` is set.

No other findings. Other callers are unaffected, since the change is test-local and no signature changed.

Further calls (2 tool calls):
1. Read `junction_pattern_integration_test.go` lines 236-260 (`containsLine` semantics), `hubforge/reanchor.go` (does `ReanchorPair` re-wire and so add the line), and `gitkit` `ExcludeLines`, in one Bash call. My context lacked all three.
2. None further.
