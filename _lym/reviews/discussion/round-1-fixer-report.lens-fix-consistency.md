```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling decisions, criteria and cross-references around it (fix-consistency lens).

**[MEDIUM] D13's new `internal/fabriccli` paragraph conflicts with `PATTERN-fabric-boundary`'s isolation rule**
- Changed text: D13, "`internal/fabriccli` is fabric's command layer and is not bound … the teardown child … and the detached push child … get their run-path, run-lock, status and takeover seams there."
- Summary: the record moves shed knowledge out of `fabricengine` into `fabriccli`, and `fabriccli` is still part of fabric. That extends fabric's shed debt instead of moving it out to shed.
- Evidence:
  - `pattern/PATTERN-fabric-boundary.md`'s Fabric isolation section says "Fabric is `internal/fabricengine` and `internal/fabriccli`" and "A change that touches it moves the knowledge out to the module, and never extends it."
  - The record's own PATTERN changes line still says "fabric is `fabricengine`, `fabriccore`, `fabriclayout` and `fabriccli` … fabric learns nothing of … shed."
  - `internal/fabriccli/teardownchild.go` imports nothing from shed today, so building the teardown child's shed seams there adds new shed knowledge to fabric.

**[MEDIUM] D13's required-seam refusal and the workspace builder's nil meaning contradict Scope Out and D5**
- Changed text: D13, "A verb whose run would use one refuses before acting when it is unset". Also D13, "with none installed a regeneration writes and removes no workspace file".
- Summary: both add a new refusal or a new skipped write to fabric's verbs. Scope Out excludes "Any change to what a fabric verb does, refuses, prints or writes", and D5 says "Every verb refuses, prints, writes and pushes the same as before".
- Evidence: neither D5 nor Scope Out was changed to except a process that leaves a seam unset. The only guard is the new `cmd/lym` criterion, and it covers the builder alone.

**[LOW] D12 narrows the admitted kits, but only boardkit gets a move rule and a guard mention**
- Changed text: D12, "`internal/testkit/boardkit`, which imports `internal/boardengine`, is outside that set". Also D12's new bullet "A test that uses `internal/testkit/boardkit` to drive the board moves…" and the D12 criterion "an import of `internal/testkit/boardkit` included".
- Summary: under the new rule, `takeoverkit`, `shedfake`, `shedcheckkit`, `stencilkit` and `envkit` are also outside the admitted set, since each imports a cut module or `boardengine`. One fabricengine test imports `takeoverkit` today, and the record gives it no move or rewrite rule.
- Evidence: `go list` imports: takeoverkit → shedengine; envkit → boardengine and shedengine; stencilkit → stencilstore. A grep of `internal/fabricengine/*_test.go` shows one import of `testkit/takeoverkit`.

Tool calls:
- One Bash call listed the testkit imports in fabricengine's test files, the imports of the candidate kits and `teardownchild.go`'s imports. My context held none of these.
