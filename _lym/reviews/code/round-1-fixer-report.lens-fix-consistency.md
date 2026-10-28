```yaml
lens: fix-consistency
outcome: reported
```

**Findings**

- **LOW**: `pattern/PATTERN-gate-slot.md:28` lists the checks in a different order than the code runs them.
  - The PATTERN line gives `Validate`, `CheckRelevant`, `CheckDecisionIDs`, then the citation check.
  - The `ValidateLoomDiscussion` doc comment at `internal/loomshed/gates.go:120` gives `Validate`, `CheckDecisionIDs`, the citation check, then `CheckRelevant`.
  - The parenthetical is a name list, not a sequence, so this is cosmetic. Reordering it would remove the mismatch.

**Checked and consistent**

- F2 matches the spec: check 55 in `contracts/specs/loom-plan-spec.md` (line 635) lets `Uses`, `Delete`, `Rename` or `Move` refs cover by file, and lets the glyph route match any group. The new `coversSeamTest` does exactly that.
- Other copies of the F1 pair: `discussionparser/validate.go:2` and `gates.go:120` already name the decision-id, citation and relevant checks, so no twin is left stale.
- Seam-test mentions elsewhere (`doc.go:28`, `plancontext.go`, the plan template) describe the check without naming the file-cover groups, so none needs an edit.
- `stripHandles` is a new helper, and `strippedRefs` keeps its old behaviour.

Tool calls:
1. One `grep`/`sed` Bash call. It read the `CheckRelevant` mentions, spec check 55 and line 100, the commit stat and the `seamtest.go` context. My context lacked the spec text and any sibling copies of the F1 sentence.
