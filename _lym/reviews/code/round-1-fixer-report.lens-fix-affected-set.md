```yaml
lens: fix-affected-set
outcome: reported
```

no findings

F1 names two branches, and the fix pins both:
- **Numbered orch role:** `ly:bm:orch-2` is the form `MatchesRole` accepts (`orch-2`, n ≥ 2), and the new passing upsert exercises it.
- **Non-string project:** the new refusals row `{"slug":"member","project":5,...}` reaches `project, _ = value.(string)`, which gives `""`. `BeamUnder("")` then fails the beam match, so the guard refuses.

Findings left unchecked: none.

Tool calls:
1. `grep -n -A12 "func MatchesRole"` over `internal/agentname`, because the context lacked `MatchesRole`'s rules, which decide whether `orch-2` is a numbered role.
