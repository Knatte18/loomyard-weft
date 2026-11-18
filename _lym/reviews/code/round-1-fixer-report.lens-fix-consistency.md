```yaml
lens: fix-consistency
outcome: reported
```

Checked the F1–F6 change set against its surrounding text and cross-references. Two siblings were left stale.

**Findings**

1. **LOW**, `internal/driftengine/deps.go:64-67`, the `Deps.DecisionRecord` doc comment.
   - **Summary:** the comment states the old, wider exemption, which F3 narrowed.
   - **Evidence:** it says "a change made only of such entries is no judge's edit". The exemption now also requires the record to be committed in the records checkout and every carry-over block to be kept. The comment also says "Empty exempts no change of it", which is still true. `doc.go` and `judgesnapshot.go` carry the narrowed wording.

2. **LOW**, `internal/loomcli/decision.go`, the `decision add` help text (the `Long` string, "--amends-plan marks a decision…" through "A decision on a PATTERN entry also passes…").
   - **Summary:** the help text omits the new F4 refusal.
   - **Evidence:** the text lists when `--amends-plan` is refused ("a recipe with no Plan-Write row and with no approved plan") and says nothing about an open Drift-Verify judge session. The same refusal applies to the PATTERN-change form, and that paragraph does not mention it either. The `decisionVerb` doc comment and `doc.go` do state it, so the operator-facing help is the one sibling that disagrees.

**Checked and consistent**

- No other source or doc text in the repo outside `_lym` and `.lym` refers to the removed refusal ("covers no line", "every line is keyed") or to the new refusal, so no other twin is left over.
- The `seats.go` format text ("covers no keyed line") still agrees with the `Expectation.Cover` doc and with the accepted F1 row.
- The removed `ParseExpectation` and `doc.go` sentences leave no dangling reference.

**Tool calls**
- `sed deps.go:58-70`, plus a grep and sed of the `decision add` help text in `decision.go`: my context lacked `deps.go` and the help-text lines, which the fixer never read.
- `grep` across the repo for the old refusal wording and the new judge-session wording: to find any untouched twin outside the files in my context.
