```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: check that the F1–F4 fix covers every member of the sets the findings name (comments describing the gated exception, tests pinning the old step line).

- F1 comment set: the old "one exception", "bar one run" and "but one run" wording is gone from `contracts/stencils/darn/darn-fragment-make-change-gated.md` (line 3), `contracts/stencils/stencils.go:55`, `internal/darnengine/spec.go:68,79-81` and `internal/darnengine/doc.go:79-83`. The grep found no other member of the set.
- F3 test set: `gatedReproduceRun` no longer appears anywhere. `gatedNoGateStep` is the same line, "3. Run no `lym gate test` yourself.", in `spec_test.go:76`, `wiring_darn_test.go:266` and the fragment (line 23). The only other copy of the item-3 text outside the fragment and its tests is the `darnwiring.go:279` comment. It says "drop its gate runs", which is correct again, as the review's Rejected section says.
- F1, `-- -run` in the darn fragment: no `-- -run` or Go test flag remains in the darn stencils. The `-- -run` hits elsewhere are `gatecli`, `gateslot`, `loomshed` and `shellguard` tests, which have nothing to do with the fragment.
- F2 and F4: they cover one place, the reproduce paragraph. It now opens item 1 and names the test and tier. It treats a pass as "no longer reproduces" only when the report shows the test ran. The rest of the fragment has no second copy of this text.

Tool calls:
1. grep for stale exception wording across `contracts`, `internal` and the docs. I needed to find leftovers beyond the files the review named.
2. grep for sibling "no gate" and "gate runs" wording in the darn stencils, `darnengine` and `loomcli`. I needed to confirm that the item-3 line and its comments agree.
