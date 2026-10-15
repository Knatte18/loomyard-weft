---
verdict: APPROVED
---

The design covers the board entry's goal and the operator decision of 2026-10-15, no more and no less.

- Follow on when a run starts: D2, through loom's start bootstrap, so a reused pair's next run re-arms it too.
- The viewer goes to the overview, then to each newest agent window, from the shell too: D2 and D3.
- A manual window change (status-bar click, Alt+Up/Down) turns follow off and the button shows it: D5 and D7.
- Clicking the button turns follow back on and jumps to the newest agent, else the overview: D7 and D8.
- The second suspected cause, attach selecting nothing while another client is attached: D4.
- The configurable button set stays with note `reed-status-buttons`, as the orch call requires; Scope Out says so and leaves that note untouched.

The click-from-on toggle (D7) is an addition the board does not state, but it rules nothing out against it, both notes already call the button the "follow toggle", and Scope In names it as deliberate.
D7's placement on the session line is one of the options `reed-status-buttons` lists as undecided; D7 states those notes may move the button, so it settles nothing for the button set.
The `batten` label carries no batten change; Scope Out explains that the fix lives in reed and loom's bootstrap, which covers both attach routes the entry names.

## Overlaps

- `kickstart-per-card` (row 10, Plan-Burler-Fix): both change `internal/loomcli` and its `doc.go` at different places, a new `card-pack` verb there against `ensureStatusStrand`'s bootstrap here.
  A text-only overlap with no conflicting decision.
  Landing order: `kickstart-per-card` is further along (row 10 against own row 3) and lands first; this run takes its changes at Merge-In.
- `report-timing` (row 3, Discussion-Write): no overlap; it changes `internal/reportengine`, `internal/reportcli`, `internal/statuspane` and `internal/shedverbs`, none of which this design touches.
  Same row as this run, a tie, which matters only if an overlap appears later.
- Parent commits: `lyx loom peers` lists none.
- `cmd/lyx/testdata/test-budget.yaml` may be shared bookkeeping with either peer; it does not escalate.

Board entries checked beside the peers: `reed-status-buttons`, `reed-status-buttons-prototype`, `engine-config-seams` (reed strands tagged by provider, a different reed seam) and `spawn-audit-log` (reed's launch path generally); none is duplicated or contradicted.
