lyx webster recover-batch refused

Arguments: 01

```
webster: batch 01 failed on findings recovery cannot check: fabric-reference: ran a fabric-referencing command ("sed -n 285,300p contracts/specs/refusal-spec.md | cut -c1-200; grep -n \"fabric commit\" -r contracts/specs plugins 2>/dev/null | head -5; lyx fabric --help 2>&1 | head -30") that can rewrite run state — an implementer fork must never touch the fabric repo directly (the finding names no path); way forward: when the worktree is clean and HEAD is the batch's start commit ("lyx webster reset --to batch-start --batch 01" moves it there if the batch committed), or the batch's commits are kept and every recorded command is read-only, "lyx webster accept-audit --batch 1" then "lyx webster recover-batch 1"; otherwise 1) lyx webster reset --to start; 2) lyx webster run
```

needs_fresh: true
