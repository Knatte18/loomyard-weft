```yaml
lens: fix-proposal
outcome: reported
```

no findings

The F1 proposal holds up. The overview clause uses `decision-<slug>.md`, which matches `internal/boardengine/decision.go:3` and `doc.go:293`. A `Sketch` link under the slug of a discussed entry matches `doc.go:206` and `doc.go:294`. I found no sibling statement the new clause contradicts.

Tool calls:
1. `grep -rn "decision-" internal/boardengine/*.go`: to check the filename pattern in the overview clause against the code, which my context did not hold.
