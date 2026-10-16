---
verdict: APPROVED
---

## Scope against the board entry

The design maps each of the entry's eight defects to a decision: #677 to D1, #681 to D2, #682 to D3, #683 to D4, #684 to D5, #685 to D6, #686 to D7 and D8, and #678 to D9.
Each decision meets the expectation its board section states:

- #677: D1 cuts a torn last line that ends in a newline, and the plan-writer note puts #677 in the first card, as the entry's "Fix #677 first" asks.
- #678: D9 prints the pane capture on failure and addresses the folder-trust candidate the entry names.
  "The smoke test passes" can only be proven by the operator's by-hand llm run, which the open risks state.
- #681: D2 puts a short error on stderr.
  Widening it to every creel refusal stays inside creel's own refusal path and is recorded as an auto-mode assumption.
- #682: D3 drops a stale stall, one of the two outcomes the entry allows.
- #683: D4 returns once the spawned daemon has exited.
- #684: D5 shows a carried message as a `carried` event, the entry's minimum.
- #685: D6 settles a removed holder within 60 s, matching C11's "about 60 s".
- #686: D7 names a reason for a pane with no input box, and D8 expires an inject at its TTL.

The out-list keeps #679 and #680 out, as the entry requires.
It also keeps out C13's stacked half, which is the note `creel-c13-stacked-rewrite`, and creel stages 2 and 3, which are `creel-notices` and `creel-agents`.
No decision does work that belongs to another board entry.
The new engine-neutral startup-gate seam in `internal/shuttleengine` (D9) serves only #678 and duplicates no entry; `codex-engine` and `creel-agents` do not plan it.
The note `strand-pane-lyx-path` (#680) is named only as an open risk for sandbox re-checks and is not worked on.

## Overlaps

- Peer `reed-add-parentless-placement` (row 12, own row 3): the peer lands first and this run takes its changes at Merge-In.
  - Overlap: D9 adds `--anchor own-window` to the parentless `reed add` in `internal/creelcli/smoke_helpers_test.go` as the #673 workaround.
    The peer's D2 keeps `own-window` an accepted value and makes it the default, so the explicit flag stays valid after the peer lands.
    The designs do not conflict.
    This run's open risk expects that fix to remove the flag, but the peer's sweep does not cover creel's smoke helper, so the flag stays as a harmless redundancy unless Merge-In removes it.
  - Overlap: the peer's D8 changes `internal/shuttleengine`'s `Display` comment, and this run adds an `IdleProbe` field and a startup-gate seam in the same package.
    These are different places in the package, so the overlap is text-only.
  - Overlap: `tools/sandbox/SANDBOX-CREEL-SUITE.md`.
    The peer's D8 sweeps only the two reed suites, and this run edits only the C4, C6, C8, C10 and C11 watch lines.
    C13's stacked half is left to `creel-c13-stacked-rewrite`, so there is no conflicting change.
- Peer `loom-stencil-wording` (row 12, own row 3): no overlap.
  Its paths are under `internal/burlercli`, `internal/loomcli`, `internal/languageconfig`, `internal/webstercli`, `contracts/specs`, `docs/overview.md` and `cmd/lyx/testdata/test-budget.yaml`, and the shared budget file is bookkeeping only.
- Parent commit `0a1f161` (Orch role: arbiter-first escalations, ORCHESTRATOR.md rendered in): no overlap.
  It changes the orch stencils, `internal/orchengine` and `internal/orchcli`, none of which this design touches.
