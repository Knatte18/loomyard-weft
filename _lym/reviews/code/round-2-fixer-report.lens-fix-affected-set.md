```yaml
lens: fix-affected-set
outcome: reported
```

Task: check whether the F14 fix covers every site that claimed operator feedback is outside the closed scope.

Findings left unchecked: none. F14 is the only finding in this review.

no findings

One search of `contracts`, `internal`, `docs/overview.md`, `docs/reference` and `_lym/landing` for exemption wording found no remaining claim that operator feedback is exempt or needs no record entry. The only remaining "outside the line" claims about this scope line concern Finalize (`orch-role.md:156`, `_lym/landing/summary.md:25`). The other hits are about unrelated things.

Tool calls:
1. One `grep` for exemption and operator-feedback wording across those paths. My context held only the parent's narrower searches, not one across all these sites for F14's claim.
