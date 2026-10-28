```yaml
lens: fix-proposal
outcome: reported
```

Findings (fix-proposal lens):

- F1 (emphasis line for `way-forward`): no case found. The `twill-plan` siblings end with list plus emphasis line, and the D1 Constraint and acceptance line hold.
- F2 (local layer can point at any fan): no case found. The first Bound bullet already admits any defined fan, so the new wording agrees with it.
- F3 (Auto-mode bullet for `review-discussion`): no case found. It matches support-log Q3.
- F4 (retired-key sentence): one finding, below.

**[MEDIUM]** D5's retired-key sentence, the clause "a hand-edited local file holding one is refused at load until a reconcile migrates it, as for every retired key".
The clause says every retired key is refused at load, and the code does not do that.
Evidence:
- `ValidateLocal` admits a raw retired key whose category is `CategoryNone` without refusing it (`internal/configengine/layers.go:276-280`: `retiredCovering` then `continue`).
- Loom's spec declares `RetiredKeys: {"fan_review": none}` (`internal/loomengine/config.go:555-557`), so a hand-edited local `fan_review` passes at load.
- `webster_reviewer_fan` and `webster_fan` are refused only because they sit in the rename lists (`config.go:1022,1035`) and not in `RetiredKeys`, so they fall through to "key %q is not in the loom template" (`layers.go:283-284`).

So the refusal is particular to these two keys, not a rule for retired keys.
Dropping "as for every retired key" fixes it.
So does replacing it with "since neither is declared a retired key of loom's spec, and `ValidateLocal` refuses it as not in the template".

Tool calls:
1. Read `retiredCovering`, loom's `RetiredKeys` lines and configsync's Migrate references, to test the "as for every retired key" claim; my context had only `layers.go:270-300` and the rename-list lines.
2. Read `internal/loomengine/config.go:553-575` to see loom's full `RetiredKeys` map, which the first grep only located.
