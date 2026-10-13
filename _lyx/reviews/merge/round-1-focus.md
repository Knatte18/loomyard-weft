---
round: 1
focus:
  - "internal/loomcli/cli.go: the task added `fmt` to `verbUsesLightweightWiring` and `c.fmtCmd()` to `parent.AddCommand`, and the parent added `c.driveCmd()` and the `spawnGoDriver` seams to the same file. Settle whether `drive` and `fmt` are each routed correctly by `verbUsesLightweightWiring` and `resolveRunID` after the merge, and whether `drive`'s run-id handling collides with the task's `fmt` exclusion in `internal/loomcli/arm.go`."
  - "internal/loomcli/wiring_test.go and doc.go: the parent extended `TestWire_ReviewKeysReachTheEnv` for hard-cap keys, and both sides added doc sections to `internal/loomcli/doc.go`. Settle whether any wiring test or doc enumeration of verbs, lightweight-wired verbs or refusal rows is now stale for one side's verb."
  - "contracts/specs/refusal-spec.md, contracts/stencils/ and internal/editdirective: the task added `fmt` refusal rows and an edit-directive line, and the parent changed stencils and refusal rows. Settle whether the stencil tests, `roleskills_test.go` and the spec's rows still agree with the merged stencils."
  - "cmd/lyx/testdata/test-budget.yaml: both sides raised rows, including `internal/loomcli`, `internal/battencli` and `contracts/stencils`. Settle whether each merged row matches the merged package's top-level test count, and whether the merge made any test redundant with another."
  - "PATTERN.md and pattern/PATTERN-driver-choice-single-site.md: the parent widened the single-site rule to the bootstrap verbs, `lyx loom drive`'s package included. Settle whether the task's new code in `internal/loomcli` (the `fmt` verb, the teardown retry in `internal/fabricengine`) reads a recorded seed driver, and whether the task's PATTERN edits survived the merge intact."
  - "internal/fabricengine (task: destroy, remove, latewriter) against the parent's `internal/shedrun/paths.go` and run-directory changes: settle whether teardown or worktree removal now interacts with the parent's driver spawn or run paths in a way neither side assumed."
---

Seed focus only; each entry names a place and a question, and the reviewer judges the merged code against the rubric's `Do not flag` list.
