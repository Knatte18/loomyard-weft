# Card 33 — goto-admissible-targets

**Edit:**
- `internal/shedengine/goto.go`
- `internal/shedengine/doc.go`
- `internal/shedengine/goto_test.go`
- `internal/shedengine/gotobudget_test.go`
- `internal/shedverbs/goto_test.go`
- `internal/loomcli/gotoentry_test.go`

**Uses:**
- `internal/shedengine/producer.go`
- `internal/shedengine/status.go`

**Intent:**
Fix PR #321 finding 1's engine half: `Goto` accepts any target from any state but done, so a run awaiting at PR-Gate can be moved onto Finalize and merged unreviewed, a run can skip Webster-Review by moving onto Describe or Publish, and a step-driven run sitting at `running` between steps can be moved under a live driver.
After this card `goto` only ever moves a halted run back, so it never skips a row's own work or a review or approval row (Discussion-Review, Plan-Review, Webster-Review, PR-Gate).

`Goto` keeps its existing refusals (lock held, missing status file, invalid state, done, unknown target) in their current order, then applies, before any write:

1. `state: running` refuses, whatever the target, because the run lock is free between a step-driven driver's steps and nothing else tells a live driver from a crashed one:
   `goto moves only a halted run, and this run is running; way forward: "lyx shed pause" then "lyx shed step" leaves the run paused at its next producer boundary, then re-run goto`.
2. The reference row is the row `current_producer` names.
   When `current_producer` names no row in `Producers` (a recipe row renamed or removed mid-run), the reference row is the producer of the latest history entry that names a row in `Producers`;
   that entry's routed row is also admitted: its producer's `OnDone` row when the entry's outcome is done, its `OnStuck` row when stuck, and nothing extra for a goto entry.
   With no such entry, only the first row of `Producers` is admitted.
3. A target is admitted when its position in `Producers` is at or before the reference row's position, or it is the routed row step 2 admits.
   `state: awaiting` narrows this to targets strictly before the reference row: an awaiting row is a hand-off, and moving onto it again or past it would bypass the hand-off.
   `paused`, `blocked` and `failed` admit the reference row itself, which is the exhausted-budget way forward (`goto --to <the blocked row>`).
4. A target that is not admitted refuses, naming the reference row and listing every admitted target in list order:
   `goto target %q lies past the run's current row %q; goto only moves a run back, so it never skips a row's own work or a review or approval row; way forward: re-run goto with --to naming one of: <admitted rows>`.
   The awaiting form says the run is awaiting at that row and names `lyx shed status` for the hand-off it waits on, before the same list.

Positions compare by list order, which puts each offshoot row (a Burler, PR-Rework) after the row it hangs off, so a run at an offshoot can return to its partner and never move past the partner's main-line successor.
Every refusal leaves the status file byte-identical.
`GotoRequest` gains no field in this card; card 34 adds the run-id and the told missing-status text.

Update `doc.go`'s goto section to state the admission rule in one paragraph and drop any claim that goto places no restriction on targets.

Tests (`goto_test.go`, untagged, status files under `t.TempDir()`, a producer list shaped like the loom recipe's tail: `Webster`, `Webster-Bouncer`/`Webster-Burler` sharing a segment, `Describe`, `Publish`, `PR-Gate`, `PR-Rework`, `Finalize`):

- awaiting at `PR-Gate`: `--to Finalize` refused, the message lists `Webster`…`Publish` and not `PR-Gate` or `Finalize`, the file unchanged; `--to Publish` admitted, leaving the run paused at `Publish`.
- blocked at `Webster-Bouncer`: `--to Describe` and `--to Publish` refused; `--to Webster-Bouncer` and `--to Webster` admitted.
- running at `Describe` with the lock free: refused naming `lyx shed pause` and `lyx shed step`, the file unchanged; after the test sets `pause_requested` and a `Step` persists paused, `goto --to Webster` is admitted.
- `current_producer` naming a removed row, latest history entry `Webster` done: `--to Webster-Bouncer` (its `OnDone`) admitted, `--to Describe` refused; with an empty history, only the first row is admitted.
- blocked at `PR-Rework`: `--to Finalize` refused, `--to PR-Gate` admitted.

Existing tests whose fixture moves a run past its own current row (`goto_test.go`'s blocked-run case, `gotobudget_test.go`, `internal/shedverbs/goto_test.go`, `internal/loomcli/gotoentry_test.go`) change their fixture's current row or target so the move is backward, keeping what each asserts; a test that asserted any-target freedom asserts the refusal instead.

**ImpactSummary:** Goto refuses running runs and every forward target; its existing refusals and the OutcomeGoto history contract are unchanged.
