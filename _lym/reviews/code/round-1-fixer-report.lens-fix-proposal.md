```yaml
lens: fix-proposal
outcome: reported
```

Findings: none.

Challenge cases built against the row: word-pair path misses, shape [2,2] matches, 2 unchanged of 4 makes `2*2 > 4` false so the break is flagged; under `>=` the row fails; `endsSemanticBreak` does not pass it early; no collision with other rows.

Tool calls: none.
