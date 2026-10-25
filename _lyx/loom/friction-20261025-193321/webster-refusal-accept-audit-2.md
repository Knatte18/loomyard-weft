lyx webster accept-audit refused

Arguments: --batch=6

```
webster: pending audit findings cannot be accepted: batch 06's commits are kept only when every fabric reference is a read-only command, and this one is not: fabric-reference: ran a fabric-referencing command ("lyx gate test --wait --gate card --card 6-clone-records-marker-on-prime-weft 2>&1 | tail -40") that can rewrite run state — an implementer fork must never touch the fabric repo directly; way forward: 1) lyx webster reset --to batch-start --batch 06; 2) re-run "lyx webster accept-audit --batch 6"
```
