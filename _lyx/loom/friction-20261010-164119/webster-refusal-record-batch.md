lyx webster record-batch refused

Arguments: 01

```
webster: batch 01-verify-log-per-run failed: webster: fabric-reference violation in "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYXHUB-quick-recipe/e67c9a3b-6efc-4de8-ad4b-f411b2b012bc/subagents/agent-a4797b1dac76dd596.jsonl": ran a fabric-referencing command ("sed -n 285,300p contracts/specs/refusal-spec.md | cut -c1-200; grep -n \"fabric commit\" -r contracts/specs plugins 2>/dev/null | head -5; lyx fabric --help 2>&1 | head -30") that can rewrite run state — an implementer fork must never touch the fabric repo directly; report archived to /home/knatte/Code/loomyard-LYXHUB/quick-recipe/_lyx/webster/reports/01-verify-log-per-run-20261010T152931Z.yaml; way forward: lyx webster recover-batch 01
```

batch: 01-verify-log-per-run
batch_failed: true
warnings: <nil>
