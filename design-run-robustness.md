# Run records tolerate newer binaries; a hung verify step is not re-run

`run-robustness` · Task · bug, batten, shed, webster · awaiting · Plan-Bouncer

Two bugs found on 2026-10-12, bundled as one small task.

## Run records and newer binaries (was `record-unknown-fields`)

### Seen (2026-10-12)

`plan-gate-fixes`' batten halted at Run-Shed after its child loom run was done, with `battenshed: Run-Shed: read status: state: decode failed: json: unknown field "routed_to"`.
The batten process had started on an older binary than the one the loom run wrote its `status.json` with, and the newer binary had added `routed_to`.
Re-running `lyx batten run plan-gate-fixes` on the new binary landed it.

The batten and loom processes of one run are long-lived and can be on different binaries, since prod moves under them at every landing.
A reader that refuses unknown fields therefore halts a run whenever a landing adds a field to a record it reads.
`board-field-drop` fixes the same class for `board.json` writers, which dropped fields they did not know, but not for readers of other records.

### Want

Every reader of a run record (`status.json` and the other whole-file records batten, shed and loom read across processes) accepts fields it does not know, and every writer of such a record keeps them, as `board-field-drop` does for the board.
A test pins it: a record with an extra field reads and round-trips.

## A hung verify step re-run (was `verify-hang-rerun`)

### Seen (2026-10-12)

In `board-field-drop`'s Webster verify gate, `TestConcurrentReadsDuringUpserts` spun at about 600% CPU (a livelock in new code), so the gate ran for minutes and loaded the machine.
The orch stopped the test binary with SIGQUIT after about 7 minutes, and verify-1 failed with the goroutine stacks.
Two seconds later verify-2 started on the same code, with no fix in between, and hung the same way until it was stopped too.
The test binary's own timeout is 10 minutes, so a hang costs at least that, at full CPU, per attempt.

### Want

- A gate does not re-run a step that hung or was killed by a timeout or signal: a hang is a failure that goes to the fixer, not a flake.
  Which code started verify-2 (a flake retry in the gate, or the Webster loop) is the first thing to find.
- A gate step that keeps the CPU busy past a budget far below the 10-minute test timeout is stopped with SIGQUIT, so the stacks reach the log early.
