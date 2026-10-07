lyx webster record-batch refused

Arguments: 8

```
webster: batch 08-calibrate-template failed: webster: fabric-reference violation in "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYXHUB-card-batching/c3d70426-a3ed-485f-b1f4-8df4b578ff73/subagents/agent-a7c787384729cc465.jsonl": ran a fabric-referencing command ("cat internal/batcher/template.yaml; ls ../card-batching-weft 2>&1 | head; go run ./tools/tokencount -last 40 -calibrate cautious -history ../card-batching-weft 2>&1 | tail -60") that can rewrite run state — an implementer fork must never touch the fabric repo directly; report archived to /home/knatte/Code/loomyard-LYXHUB/card-batching/_lyx/webster/reports/08-calibrate-template-20261007T130030Z.yaml; way forward: lyx webster recover-batch 08
```

batch: 08-calibrate-template
batch_failed: true
warnings: <nil>
