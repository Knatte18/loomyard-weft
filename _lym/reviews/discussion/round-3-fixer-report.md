# Round 3 fixer-report — discussion

## Changes

- **F1 (MEDIUM), fixed differently in one part.**
  Decision 1 replaces the false "today a gone worktree only fails each tick" with the `os.MkdirAll` recreation (`internal/state/state.go` `WriteJSON`, `ReadJSON`), and tests the worktree's `.git` link at `Geometry.WorktreeRoot` at the top of every loop iteration, not only on a probe tick.
  It states the bound that a tick in flight at the removal can leave an empty `.lym/beam` stub, and that the exit's own writes create nothing there.
  The Acceptance criterion now says the exit fires at the top of the next loop iteration, and a new criterion covers a dobby woken by a file event after the removal, which still exits and stops a tidy held in `State.Tidy`.
  Not made as proposed: the kept `State.Tidy` is "the last non-nil one read while the marker existed".
  A tidy settled before the removal sets `State.Tidy` to nil (`settleTidy` in `internal/dobbyengine/tidy.go`), so a last-non-nil rule would keep and stop a tidy that had already ended.
  The record instead keeps the `State.Tidy`, nil included, of the last tick that ended with the marker present, and bounds the in-flight tick's tidy.
- **F2 (LOW), fixed as proposed.**
  Decision 2 names the re-exec test once: this pid and either this process's start ticks or none.
  The lock wait and every notice of Decision 3 gated on the test treat the no-ticks record as a re-exec.
  `recordStart` is the one use that tells it apart, giving it a full pass and writing the ticks.
  The Acceptance criterion on the no-ticks record adds that a refused re-exec or a wiring failure on it posts its notice.
- **F3 (LOW), fixed as proposed.**
  Decision 5's lock-holder rule is scoped to the files it governs, and the Windows paragraph names the spawner's `dobby.out` rotation as the one exception, bounded by reading the lock free through `Live`, with the Bound's rename race as what it leaves.
- The support log gains a Round 3 entry under Review rounds.

## Self-check

- fix-consistency, LOW: the lock-holder rule scoped to `dobby.log` alone left the Windows `dobby.out` exception with no rule to depart from. Fixed as proposed: the rule now covers `dobby.log`, and `dobby.out` where `selfexec` execs, and the Windows sentence points at that rule.
- fix-proposal, LOW: the no-ticks record counted as a re-exec makes a pid-reusing fresh start post an undeserved notice, with no bound stated. Fixed as proposed: Decision 2 states that bound.
- fix-proposal, LOW: the in-flight tick's bound named only the stub, not its passes, task starts and tidy step over the empty `State`. Fixed as proposed: the bound names them.
- fix-way-forward, LOW: the kept-tidy rule needs a marker test when a tick ends, which Decision 1 did not state. Fixed as proposed: the dobby tests the marker again as each tick ends and replaces the kept value only when the marker is there.
- fix-claims, LOW: "removes nothing and returns nil" for the second stop of a settled tidy is conditional in `removeStrandIfPresent`. Fixed as proposed: the bound says it removes a strand only when reed still tracks it, and logs a failed stop when reed's status probe fails.

## Disputed

None.

## Deferred

None.
