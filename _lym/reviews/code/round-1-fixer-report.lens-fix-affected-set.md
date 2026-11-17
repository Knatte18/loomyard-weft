```yaml
lens: fix-affected-set
outcome: reported
```

No finding. The only other site that names the sibling check ids is the spec's check list, and the fix added `label-duplicate` there, so the check-id set is covered.

Findings: no findings.

Unchecked: none.

Tool calls:
1. `grep` for `label-unknown`, `CheckLabelUnknown`, `labels-missing` and `CheckLabelsMissing` outside `internal/raddleparser/check*`. I needed to know whether another doc, stencil or check-id enumeration listed these ids and would need the new one. My context held only the three files the fixer edited.
