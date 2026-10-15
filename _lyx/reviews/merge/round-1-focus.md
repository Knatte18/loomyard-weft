---
round: 1
focus:
  - "internal/loomcli/arm.go, cli.go: the parent's `card-pack` verb (lightweight wiring, resolveRunID exemption, cardPackCmd) goes through `armAt`, which now sets `c.caller` from the strand variable and `specFor` copies it onto the Spec. Settle whether `card-pack` reaches `specFor`/`Spec.Caller` consistently with the other lightweight verbs, and whether the new cardpack.go and the task's caller labelling assume nothing the other broke."
  - "internal/loomcli/drivewiring.go and the parent's loomcli files (smoke_driverstrand_test.go, smoke_drivingsurface_test.go, wiring.go, start.go): the task changed `shedverbs.RunUntilStop`/`RunGoto` signatures (caller argument, `driverStep`/`driverGoto`, `shedrun.SettleHandBack`). Settle whether every parent-added caller or test double of those seams, and of `sheddriver.Deps`, matches the task's new signatures and semantics."
  - "pattern/PATTERN-pane-binary-resolution.md (parent added the status bar follow button, `@lyx_follow_command`, `panebin.go`): settle whether any task code or test that composes or pins a `lyx` key binding or pane command follows the amended rule."
  - "cmd/lyx/testdata/test-budget.yaml, docs/overview.md, contracts/specs/refusal-spec.md: both sides edited these. Settle whether the merged budget rows still cover the tests each side added in `internal/loomcli` and `internal/reedengine`, and whether the merged docs and refusal spec describe both sides' behavior without one overwriting the other."
  - "Test economy: settle whether the task's new `internal/loomcli` tests (drivewiring_test.go, cli_test.go) and the parent's (cardpack_test.go, smoke tests) now duplicate each other, and whether `card-pack` under the caller labelling has coverage."
---

Focus is seeded from the overlapping files and packages; the diffs of both sides were skimmed, not judged.
