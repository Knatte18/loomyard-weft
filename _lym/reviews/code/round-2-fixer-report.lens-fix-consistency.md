```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the F10 change set agrees with its surroundings.

no findings

Tool calls:
- grep for mentions of the test file and for the other git seams: I needed to find sibling descriptions elsewhere, and my context did not show them.
- Read `_lym/landing/summary.md`, grepped the stand-ins inventory rows, and grepped `landingshed`'s uses of the other git seams: the summary and inventory text, and which files call those seams, were not in my context. Nothing there is made stale or contradicted by the change.
