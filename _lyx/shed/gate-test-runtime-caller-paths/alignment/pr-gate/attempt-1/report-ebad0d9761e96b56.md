---
verdict: APPROVED
---

## Alignment at PR-Gate: gate-test-runtime-caller-paths

The measure is board entry `gate-test-runtime-caller-paths` (issues #807 and #808), whose operator decision of 2026-10-22 rules: keep `-trimpath`, document it and surface it in the gate's output.
The decision record carries no `Added after Discussion` entry, so the board body stands unamended.

### Diff against the entry

Pull request 826 at `12632f188` changes only `internal/gatecli`.

- Surfacing: `trimpathLine` in `internal/gatecli/tiers.go` builds a line naming the tier, its effective GOFLAGS and the effect on `runtime.Caller`.
  `runTier` in `tierrun.go` writes it into the tier's log before the first command, and `report` in `run.go` prints it on stderr before the summary line.
  This meets the operator decision's "surface it in the gate's output" and issue #808's ask that gate output name the effective trimpath/GOFLAGS state.
- Documenting: `internal/gatecli/doc.go` gains a `# Trimpath` section saying why `-trimpath` stays and what it does to `runtime.Caller`.
- Keeping `-trimpath`: no GOFLAGS, `gate.yaml` or `gateslot.HolderEnv` changes, as the operator decision requires.
- Tests: `TestTrimpathLine` covers the detection rows, and the scenario and scopes integration tests pin the line in stderr and the log, once per tier run, and its absence for an empty scope.

Nothing in the diff goes beyond the entry, and no point of the entry is left out.

### Open risks

- The record's own risk, a `-trimpath` passed on a tier's command line rather than in GOFLAGS getting no line, is outside the entry, which names the GOFLAGS path the operator identified.
- The Darn-Review carry-over `empty-scope-no-line-untested` is closed by commit `17bdf708e`, which adds the empty-scope assertion in `scopes_integration_test.go`.
  It names no parent correction.

### Overlap with peers and parent commits

`lyx loom peers` lists no parent commits.

- `run-process-ownership` (row 10): no changed paths yet; its decision record mentions `lyx gate test` only as an example of an agent's foreground process, and changes no gate output or GOFLAGS. No overlap.
- `report-timing-unreadable-journal` (row 16): changes `internal/reportcli`, `internal/sleepjournal` and `tools/tokencount`. No overlap.
- `darn-hard-cap-default-2` (row 0): changes `internal/darnengine`, `internal/loomcli`, `internal/runprofile`, the orch stencil and `docs/overview.md`. No overlap.

No overlap exists, so no landing order applies.

### Superseded

None: the entry already absorbed issues #807 and #808, and no other board entry covers this work.
