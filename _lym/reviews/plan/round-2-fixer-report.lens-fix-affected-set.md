```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun F1's search over cause enumerations in `internal/loomcli/doc.go` and check that the fix covers every one.

Unchecked findings: none. F1 was the review's only finding, and I reran its search.

no findings

`internal/loomcli/doc.go` enumerates the arbiter's admitted causes in exactly two places: line 471, "pending escalation of cause `hard-cap`, `circling` or `decision`", and line 577, "pending hard-cap, circling or decision escalation only". The changed Intent in Card 11 names both. The file's other circling mentions (lines 292–294, 330 and 521) are about the circling verbs, the hard cap or a circling decision. They don't list the arbiter's causes, so they fall outside the set.

Tool calls:
- I grepped `internal/loomcli/doc.go` for circling, arbiter and stalemate. The fixer's earlier grep searched only for "hard-cap", so it could have missed an enumeration worded differently.
