# Arbiter relaunch, Webster deviations list, alignment landing order

`arbiter-fixes` · Task · bug, enhancement, loom, shed, webster, orch · running · Batchifier

Fix three rough edges around the arbiter and the alignment agent: a relaunch refused although the verdict is already in effect, a Webster deviations list that mixes in the batch's own files, and a landing order the alignment agent should settle itself.
This task builds on the driver and arbiter changes of `go-driver-assistant` once they have landed.

## arbiter-relaunch-moved

`operator-followups`, 2026-10-12: Merge-In verify failed after merging the parent, and the driver ran the arbiter (attempt 2).
17 seconds after the arbiter spawned, the run moved from Merge-In's `stuck` row to Webster-Burler-Review, the same row the arbiter's verdict later named as its goto.
The relaunch was then refused with bootstrap "the run moved since the arbiter spawned", and the run parked for the parent although the verdict was already in effect.

### What to fix

- A relaunch whose verdict goto names the row the run already stands on should be admitted, not refused on the stop-key mismatch.
- Find what moved the run during the arbiter's spawn: a second actor moving a run while its arbiter is live is a race of its own.

### Seen again (2026-10-13, `reed-multi-window`)

The run stopped at Merge-In on a verify failure (a test-budget row), the status file recorded a goto to Webster-Burler-Review at 19:04:25Z, and arbiter attempt 2 then answered `resume` with that same row as its goto.
The driver's re-launch was refused with kind `bootstrap`, "the run moved since the arbiter spawned", although the verdict's goto target was the row the run already sat on.
A `lyx pause` (19:04:42Z) and an overnight host sleep fell inside the arbiter's spawn, so the pause or the goto ahead of the spawn may be what counts as the move; the orch re-launched by hand.

### Also seen

The same arbiter got an invalid-JSON error from `lyx board get operator-followups`: it likely passed a bare slug, not a JSON payload; check the arbiter stencil's wording.

## webster-deviations-list

### Seen (2026-10-12, `pause-esc-rewind`)

The Webster report `_lyx/webster/reports/01-running-turn-predicate.yaml` listed six `deviations`: `interruptguard.go`, `send.go`, `engine.go`, `fakes_test.go` and `run_inject_test.go` in `internal/shuttleengine`, plus `pattern/PATTERN-guarded-interrupt.md`.
The batch most likely held cards 1 to 5, so several of these probably house the batch's own target glyphs and are no deviation at all; the pair was torn down before that could be checked.

### Cause

The implementer stencil (`contracts/stencils/webster/webster-body-implementer.md`, the `deviations` paragraph) contradicts itself.
It asks the fork to list the paths it changed outside the batch's target-glyph union, then says the fork reports what it touched and never estimates the comparison itself, because `internal/planglyph`'s `ScopeGuard` makes it.
A fork cannot map a glyph to its file without that comparison, so it guesses, and the list mixes real deviations with the batch's own files.

### Idea

Pick one meaning:

- the fork reports every path it changed, and `ScopeGuard` alone decides which are deviations; the Merriam digest and the PR narrative show the guard's result, not the fork's list;
- or the fork reports nothing, and the guard's output over the batch delta is the deviation list.

Either way the list a reader sees comes from the guard, never from a fork's guess.

## alignment-landing-order

The alignment agent escalates to the orch on cross-run overlap with "which run lands first?".
The orch answers by one rule nearly every time: the run furthest along lands first, and the other takes its changes at Merge-In.
On 2026-10-12 it woke the orch twice for such overlaps (`cleanup-1012` and `convergence-plan` against other runs in flight), and both were settled by that rule with no design call.
The facts the rule needs are already readable to the agent: the board, `lyx loom peers` and each peer's decision record.

### Idea

The alignment agent applies the landing-order rule itself and records the outcome in its report.
It escalates to the orch only when the designs conflict: both runs change the same behavior in different ways, or a recorded decision in one contradicts the other, so a parent decision is needed in the later run.
A text-only overlap (same file, different lines or an import and a mount on each side) is never a reason to escalate.

The operator agreed the orch should get only stops that need a parent choice (2026-10-12); see the note `arbiter-when` for the matching rule on arbiter spawns.
