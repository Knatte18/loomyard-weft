lyx webster record-batch refused

Arguments: 06

```
webster: batch 06-fabric-form failed: webster: fabric-reference violation in "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYXHUB-cli-surface/3b48aefb-b68c-4aae-941c-ea8eb2256f6e/subagents/agent-a5d0d1f0ba840bec5.jsonl": ran a fabric-referencing command ("for v in \"\" add list remove checkout pairs reconcile prune cleanup unwire shortname clone status commit push pull sync diff merge merge-in merge-stage; do printf \"%s: \" \"$v\"; lyx fabric $v --help --json 2>/dev/null | jq -r .long | wc -c; done") that can rewrite run state — an implementer fork must never touch the fabric repo directly; report archived to /home/knatte/Code/loomyard-LYXHUB/cli-surface/_lyx/webster/reports/06-fabric-form-20261010T144105Z.yaml; way forward: lyx webster recover-batch 06
```

batch: 06-fabric-form
batch_failed: true
warnings: <nil>
