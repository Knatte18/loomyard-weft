---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: go-driver-assistant decision record, section Added after Discussion (parent, 2026-10-12)
    summary: This run shares Go and stencil files with the in-flight go-driver-assistant run, whose parent-recorded decision says go-driver-assistant lands before judge-cap-call, so approving here would land them in the opposite order
    fix: The parent decides the landing order. Either judge-cap-call lands now and go-driver-assistant, still at Webster, takes the shared files at its own Merge-In, which amends that recorded decision. Or judge-cap-call waits at PR-Gate until go-driver-assistant lands and then takes the shared files at a fresh Merge-In, as recorded.
---

PR 600 at `2db7d9387` was checked against the board entry `judge-cap-call`, the decision record (D1 to D9, the added hub-default route decision, the open risks and the Discussion-Review carry-over) and the plan's nine cards.
The change stays within the entry's goal and the record's Scope.
It covers the judge's fix tag, the soft-cap rule in `shedadapters/bouncer.go`, the per-perch hard caps in loom and darn config with their load refusals, the `hard-cap` cause, the `circling continue` refusal, and the arbiter and orch stencil wording.
The arbiter's eligibility list (`internal/loomcli/arbitereligibility.go`), the driver stencil and the Go driver are untouched, as Scope Out and `arbiter-when` require.
I found no scope deviation and no break of a recorded decision within this run.
The merge of `58534d414` is a `main` commit already on `main`.
`lyx loom peers` lists no parent commits.

### [BLOCKING] Landing order against go-driver-assistant contradicts its recorded decision

**Location:** go-driver-assistant decision record, section "Added after Discussion (parent, 2026-10-12): D15 accepted"

**Issue:** `lyx loom peers` lists `go-driver-assistant` (running, at Webster) changing these files, which this run also changes:

- Go: `internal/loomengine/config.go`, `internal/loomengine/config_test.go`, `internal/loomcli/wiring.go`, `internal/configreg/runsettings_test.go`, `contracts/stencils/stencils.go`, `contracts/stencils/claims_test.go`.
- Config template and stencil: `internal/loomengine/template.yaml`, `contracts/stencils/loom/loom-template-arbiter.md`.
- Spec: `contracts/specs/refusal-spec.md`.

The parent already settled this overlap in the peer's record: "go-driver-assistant lands before judge-cap-call; judge-cap-call takes its arbiter-stencil, loom-config and loomcli/doc.go changes at Merge-In", on the grounds that the two designs do not conflict and "the run further along lands first".
Since then, judge-cap-call has reached PR-Gate while go-driver-assistant is still at Webster.
The recorded order and its stated rationale now point opposite ways.
An approve here lands this task under `alignment_pr_approve` and breaks the recorded order.
Waiting follows the recorded order but holds a finished run behind one still being built.
The board and records do not settle which of the two applies now.
`alignment-landing-order` (Ready, unbuilt) would let the alignment agent apply furthest-along-lands-first, but it is not in force.

The designs themselves do not conflict.
The peer's arbiter-stencil change (D14, the case-brief marker) and this run's D7 review rule touch different parts of `loom-template-arbiter.md`.
The peer keeps soft-cap `budget` escalations going to the arbiter until this task and `arbiter-when` narrow them, which matches this run's Scope Out.

Shared bookkeeping, not an overlap: `cmd/lyx/testdata/test-budget.yaml`, which both runs change.
