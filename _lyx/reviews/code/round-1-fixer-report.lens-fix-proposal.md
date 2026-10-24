```yaml
lens: fix-proposal
outcome: reported
```

No findings.

I tried to break F1's proposal in three ways:
- **Collision:** `PATTERN-old-hub-rule` appears nowhere else in the repo (outside `.git`), so it does not match a real or removed PATTERN ID. The new row also differs from the sibling `PATTERN-old-rule` on line 259, so no two fixture rows share an ID.
- **Parser coupling:** `internal/discussionparser/*.go` has no reference to either ID, and the package test passes after the change (per the parent's gate run). The parser does not check IDs against the PATTERN overview.
- **Row coverage:** the line keeps its `— changes:` shape, so the `changes` action is still exercised.

Tool calls made:
1. `grep -rn "PATTERN-old-hub-rule"` across the repo: my context lacked any search for ID collisions.
2. `grep -n` of both IDs in `internal/discussionparser/*.go`: my context lacked the parser source, so I could not tell whether it looks IDs up.
