# Card 13 — teardown-origin-snapshot-and-claim-mode

**Edit:**
- `internal/pairteardown#Request`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown#Teardown.settleBoardClaim`
- `internal/pairteardown#TestSettleBoardClaim_RunningBattenRecordWayForward`
- `internal/pairteardown#TestTeardown_CallSequence`

**Intent:**
Give `RemovePair` the two things the cancel's `--remove` needs (D5) and reword the held-gone warning (Scope: way-forward texts).

`Request.Snapshot` now applies in origin mode too: `RemovePair` takes the snapshot whenever `Snapshot` is set, local or not, before the removal, so a forced origin-mode removal keeps the pair's uncommitted paths under the local snapshot tag instead of discarding them; a refused snapshot returns its error with nothing removed and `AfterSnapshot` not run, as in local mode.
No existing caller passes `Snapshot` outside local mode (`teardownRequest` passes none, and the origin seams' force case is local), so only the cancel reaches the new branch.

`Request` gains `LeaveClaim bool`: with it set, a successful origin-mode `RemovePair` reads no landed answer and settles no board claim, leaving the claim to the caller, which clears it under its own locks (D7).
What it lets a caller skip: the claim settlement, which only the cancel skips, and which it replaces with its own clear of a run-form status.

`settleBoardClaim`'s held-gone warning, for a `running` batten record with no live registered batten, now names `lyx batten cancel <slug>` as the way to end the run and release the claim, in place of `lyx board set-status <slug>`; the live-batten warning keeps `lyx batten pause <slug>`.
`TestSettleBoardClaim_RunningBattenRecordWayForward`'s gone row now wants `lyx batten cancel slug` and not `lyx board set-status`.
`TestTeardown_CallSequence` gains rows: an origin-mode `Snapshot` request runs `snapshot` before `archive` and `remove` and carries the tag on the result; an origin-mode snapshot refusal makes no `remove` call; a `LeaveClaim` request with a run-form claim makes no `landed`, `readClaim`, `battenRun` or `write=` call anywhere in its sequence, the landed read before `archive` included.
The `pairteardown` package doc's Board claim and Local mode sections are reworded by card 14 together with the cancel phase.

**ImpactSummary:** `RemovePair`'s callers are batten's Worktree-Teardown row, `Run` and the origin seams; none sets the new field or an origin-mode Snapshot, so their behaviour is unchanged, and only the held-gone warning's text changes for them.
