---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-hubgeom-gate-slots.md
    summary: Card 5 adds the exported hubgeom.GateSlots constructor but leaves internal/hubgeom/doc.go, whose contract sentence enumerates the package's constructors, untouched and now false; a doc.go sentence left false recurs for a third round, so the gap is in the plan's doc sweep
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/22-webster-review-failure.md
    summary: The failure-record Bound lists what an edited record still lets through but omits the log path, which can point the reviewer at any pair-writable file in the verify directory
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/21-round-gate-tier3.md
    summary: Card 21 puts the git-reading rows of its publishfailure test in the integration tier but creates only the untagged publishfailure_test.go, naming no integration-tagged file for them
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/11-status-gate-waits.md
    summary: Card 11 reports every gateslot wait record but only the one verifytree marker per worktree, and never states why one Go-side verify wait per worktree is all that can occur
---

### [MEDIUM] hubgeom's package doc goes stale under card 5

**Location:** `_lyx/plan/05-hubgeom-gate-slots.md`

**Issue:** Card 5 creates `hubgeom.GateSlots`, a new exported constructor that cards 7, 8, 9, 10 and 11 call.
`internal/hubgeom/doc.go` reads: "hubgeom's contract today is ReedGeometry, BurlerGeometry, and WebsterGeometry, converting a Location into a reedengine.Geometry, a burlerengine.Geometry, and a websterengine.Geometry respectively", then names `ReconcileGeometry`.
With `GateSlots` added, that enumerated contract is incomplete, and no card edits `internal/hubgeom/doc.go`.
The decision record's Scope requires "the touched packages' `doc.go`" in the same commit, as does `CLAUDE.md`'s docs rule.
Every other package whose behavior a card changes has its `doc.go` sentence covered: card 14 and card 15 rewrite the card-gate, batch-wide-verify and FAILED-condition sentences of `internal/websterengine/doc.go`; cards 4, 9, 12, 17, 18, 20, 21 and 23 cover theirs.

This is the third round in a row that finds a `doc.go` sentence left false: `internal/websterengine/doc.go` and `internal/loomshed/doc.go` in round 1, the FAILED sentence in round 2, and `internal/hubgeom/doc.go` now.
Each earlier fix patched the named sentence, not the method that keeps missing them.

**Fix:** Add `internal/hubgeom/doc.go` to card 5's `**Edit:**` list.
Its Intent then states that the contract paragraph gains `GateSlots`: the one constructor of a hub's gate-slot pool, over the board dir's slot directory, with limits read from the board dir's `gate.yaml` at each acquire.
Fix the method as well: for every card that adds an exported symbol or changes observable behavior in an existing package, grep that package's `doc.go` for sentences that enumerate or describe the touched surface, and list the file on the card wherever one goes false.

### [LOW] The failure-record Bound omits the log-path redirect

**Location:** `_lyx/plan/22-webster-review-failure.md` (Bound paragraph), and the matching Bound in `_lyx/plan/21-round-gate-tier3.md`

**Issue:** Card 22's Bound says that after `checkedPublishFailure`'s shape checks, an edited record can "only add a well-formed test name that never failed … or point the reviewer at another real merge commit's diff; no shell text or free prose reaches the prompt".
The record also carries the log path, which card 21's check only confines to the verify directory.
The rubric section tells the reviewer to read the log for each failure's cause.
That directory sits in the pair's own `.lyx/`, so an edited record can point the reviewer at any file there, including the log copy itself, which a pair session can rewrite.
The prompt stays free of free prose, but the file the reviewer is told to read does not.
The rubric's attack-surface criterion asks a routing edge's card to state what it lets through, and this route is missing from the list.
The bound behind it is already in the card: Publish reruns both verifies in full regardless, and a reviewer already reads pair-authored content in the branch range.
So this is a gap in the stated bound, not an unbounded hole.

**Fix:** Extend the "what still passes" sentence in card 22's Bound, and card 21's to match, with the log path.
An edited record can point the reviewer at another file under the verify directory, and the log content is pair-writable.
Both stay bounded by Publish's full rerun.

### [LOW] Card 21 names no file for its integration-tier rows

**Location:** `_lyx/plan/21-round-gate-tier3.md`

**Issue:** Card 21 creates only `internal/loomshed/publishfailure_test.go` and calls it table-driven.
It then says "its git-reading rows sit in the integration tier".
An untagged file cannot hold integration-tier rows.
The rows that need a real merge commit (a full hex name resolving through `gitrepo` to a commit with two or more parents, and a non-merge commit dropped) need a scratch repository, which this repo builds under the `integration` tag; `internal/websterengine/doc.go` states that rule for git-behavior tests.
No integration-tagged file is named for those rows, so the implementer must choose a file that is in no card's targets.

**Fix:** Name the file.
One option is to add `internal/loomshed/publishfailure_integration_test.go` to card 21's `**Create:**` for the git-reading rows and keep the shape-only rows untagged.
The other is to state that those rows join `internal/loomshed/verifygate_integration_test.go`, which card 21 already edits.

### [LOW] Card 11 leaves the one-marker premise unstated

**Location:** `_lyx/plan/11-status-gate-waits.md`

**Issue:** D6 requires `lyx loom status` to report, for its worktree, each waiting gate with its wait start.
Card 11 does this for every live `gateslot` wait record: `lyx gate test`, the card-verify rerun and the plan gate's type load.
Go-side `verifytree.Verify` waits are reported only through the single running marker per verify directory (`verifytree.NewPaths(...).Marker`).
A second concurrent `Verify` in the same worktree overwrites that marker and removes it on exit, so its wait would not show.
In practice the plan-verify sites of one worktree run sequentially within one loom run, and only an operator's concurrent `lyx webster verify` could overlap.
The plan never states this premise, so a reader cannot tell whether the single-marker form meets D6's "each waiting gate".

**Fix:** Add one sentence to card 11's Intent saying that the Go-side verify sites of one worktree run one at a time within a loom run, so the one marker carries every Go-side wait that can occur.
It also says that an operator's concurrent `lyx webster verify` is outside that guarantee, and that this is pre-existing marker behavior.

## Focus directive verdicts

- Wait-directory geometry: no defect.
  Card 5 adds `Geometry.GateWaitDir`, filled by `hubgeom.WebsterGeometry` with `gateslot.WaitDir(anchorPath)`.
  Card 6's `RecordBatch` reads `deps.Geom.GateWaitDir` and passes it to `rerunCardVerifies`, and card 6's `**Uses:**` names no `gateslot.WaitDir`.
  No `internal/websterengine` code joins a path from `AnchorRoot`.
  The remaining `gateslot.WaitDir` calls sit in CLI layers: `loomcli` in cards 9 and 11 and `gatecli` in card 10.
  The plan stays within `PATTERN-told-geometry`.
- Attack-surface bounds for routing edges: cards 21 and 22 each state what a deleted or edited record lets through and what bounds it, apart from the log-path gap raised as F2.
- Package docs in the same commit: the FAILED-condition sentence in `internal/websterengine/doc.go` ("a fork returns FAILED only on a non-zero build/unit gate or a non-zero per-card `verify:`") is restated by card 15, which also rewrites the "no batch-wide verify" sentence; card 14 rewrites the card-gate paragraph.
  The one doc left false is `internal/hubgeom/doc.go` (F1).
- Status display of waits: card 11 reports every `gateslot` wait record and the verify marker.
  The single-marker premise is unstated (F4).
