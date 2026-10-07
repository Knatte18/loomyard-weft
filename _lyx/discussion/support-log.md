# Support log — run-recovery

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry `run-recovery` and the related entries `shuttle-turns`, `test-tiers`, `orch-notice-channel`, `runs-overview`, `orch-owns-the-loop`, `reed-status-tabs`, `batten-windows`, `gate-checklist-pane`; fixed the scope boundaries listed under Scope → Out.
2. Gate in status: found the verify running marker and loom's `Waiting` hook already exist; #420's correction shows the `bugfix-webster` blank was the background-shell wait, so the gap is shuttle's own post-turn waits, plus elapsed time and the command in the note.
3. Gate on screen: shuttle owns the gate and the shell wait, so it sets the pane mark; Publish/Finalize run in the driver's step with no agent strand, so their mark goes on the driver strand through a told callback.
4. #420: `verifyrun`'s duration is only `WaitDelay`; nothing bounds the gate. Picked a process-group kill plus a fixed deadline, with a timeout as a failed gate.
5. #408: chose dormant matching for `--unless-name` over running `reed resume` in the folder-open chain.
6. #409: reed cannot import shuttle; chose a generic done-when path list on the strand, filled by shuttle with its output files.
7. batten: a halted child's parked driver comes back through reed resume of the pair; a running child with a dead batten's spawn confirmation is spawned again.
8. Orch resume: chose `lyx loom resume` in the task worktree, waking only a live parked driver.

## Rejected alternatives

- Status from a `#()` shell call in the tmux status line reading the markers: one source of truth, but runs a lyx process per refresh per client and widens binary resolution beyond `panebin.go`.
- A session-level wait option instead of a pane option: simpler for Publish, but departs from the operator's stated design and loses the strand name.
- A step-level watchdog goroutine logging the current gate (#420): the status note and the pane mark already answer it.
- A `verify_timeout_min` config key: no one asked to tune it; it would need one key readable from loom, landing and standalone webster.
- Treating a verify timeout as a returned error: it would halt the run instead of letting the writer fix a hanging test.
- `reed resume` before the add in the folder-open chain (#408): it would revive every dormant strand, the orch's paid session included, on every folder open.
- reedcli consulting shuttle run directories at resume (#409): couples reed's CLI to shuttle's run-dir layout; the done-when list keeps reed self-contained.
- Removing a finished run's strand only from shuttle's next Attach (#409): the next step may belong to another producer and never attach to the old run.
- Having batten resume the halted child itself: the operator wants the halt to stay the child's own.
- A `lyx batten resume <slug>` verb from the prime: batten needs no wake-up, and the orch already runs pair-side verbs like `lyx loom approve`.
- Letting `lyx loom resume` spawn a Go driver: it would read the seed's driver a second time (`PATTERN-driver-choice-single-site`).

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Which gate path skips the status note? Answer: none of the verify paths; the blank was shuttle's background-shell wait (#420 correction). D1 adds a shuttle wait marker.
- Q2 (auto-pick): Where does Publish/Finalize's on-screen mark go? Answer: the driver strand, through a told callback; Go-driven runs show status only.
- Q3 (auto-pick): Deadline value and source? Answer: a 60-minute constant in `verifytree`.
- Q4 (auto-pick): Timeout as failure or error? Answer: failed gate with findings.
- Q5 (auto-pick): #408 fix shape? Answer: `--unless-name` matches dormant strands.
- Q6 (auto-pick): #409 fix shape? Answer: done-when path list on the strand, set by shuttle to the output files; resume drops a finished non-live strand.
- Q7 (auto-pick): What does batten restore per child state? Answer: running with a dead batten's confirmation → re-spawn; halted or awaiting with no live driver → reed resume of the pair, once per episode.
- Q8 (auto-pick): Resume verb home and name? Answer: `lyx loom resume` in the task worktree.
- Q9 (auto-pick): What does "records the decision" mean? Answer: the resume's own trace; no new record type.
- Q10 (auto-pick): Elapsed time in tmux format — fallback if arithmetic on `%s` does not expand: show the start time. Left as an open risk for the plan to verify live.
