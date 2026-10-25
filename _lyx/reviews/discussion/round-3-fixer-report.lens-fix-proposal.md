```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I checked each fix that went in as proposed (F1–F5, and the parts of F6 taken from its fix key) for a concrete case where the proposal itself is wrong.

1. **MEDIUM. F1, D4's new sentence "Dobby runs no pass of a run while a pause entry covers the hub or the run's slug…" and the matching clause in D4's "Dobby runs a pass … only while…" sentence.**
   - Summary: not passing a covered run also drops the per-check work that runs under a pause today. The concrete case is an idle child's parent lock: it is never released while the pause stands, so a sibling run's Finalize waits on it and fails.
   - Evidence: `internal/battenshed/innerrun.go:700-723`. `check()` runs `checkChild`, `ownerStep`, `lockStep` (on a due probe) and `noticeStep` on every Call, whatever `coverage` reports. Only the spawn hold and the Done-under-pause yield depend on coverage. So under a pause, today's long InnerRun still releases an idle owner's parent lock after the idle bound and still sends the halted-child, quiet and api-error notices.
   - Case: the operator pauses slug A only. A's child sits idle holding the parent lock. Sibling B, not covered, reaches Finalize and waits on the parent lock. B's wait has the ten-minute bound D4 names for Worktree-Create and Worktree-Teardown. No pass of A runs, so `lockStep` never releases A's lock and B fails. Every once-per-episode notice of A is also held until the lift.
   - The record says nothing about this. The acceptance bullet "an idle owner's parent lock is released after the idle bound across passes" is now false for a covered run.
   - Possible fixes: let a covered run get a probe-tick pass that skips pre-run's `refuseCovered` (the alternative the reviewer allowed, which needs the `battenPreRun` change named), or state that a covering pause holds the parent lock and the notices as an accepted bound.

2. **MEDIUM. F4, D2's widened refusal "refuses, naming `lyx adopt &lt;parent&gt;` … unless the same call adopts the parent", together with D2's beam-side refusal ("refuses while a member that is not done records another machine as its owner, naming those members").**
   - Summary: the way forward each refusal names is itself refused, now for beams as well as tasks.
   - Case: beam P and its member T are both owned by machine A. On machine C, `lyx adopt T` is refused, naming `lyx adopt P`. Then `lyx adopt P` is refused, naming T. Only `lyx adopt P T` passes.
   - Once the refusal covers beams, the chain recurses up to the topmost ancestor owned elsewhere. For example, `lyx adopt --prime` needs every not-done prime member owned elsewhere in the same call, and each of those beams needs its own members. The refusal names a single command, so an operator who follows it hits a second refusal.
   - Evidence: D2 line 61 as changed, D2 line 62, and the acceptance bullet "adopting the beam and those members in one call is not". The refusal should name the one combined call, `lyx adopt &lt;parent&gt; &lt;slug&gt;…`, with every slug the chain needs.

3. **LOW. F6, D5's commit-point rule "unless … the write is a release", where a release is "a write that sets `status` to null on an entry that was already a task".**
   - Summary: under the store rule, any internal writer that clears a status with null counts as a release, and the task becomes startable. The rule therefore only holds while every internal clear path writes `draft`. D5 names cancel and `claimClear`, but nothing states that every non-operator nil write must become `draft`.
   - Evidence: `boardengine` `SetStatus` is reached from `internal/boardcli/cli.go:656` and from the merge at `internal/boardengine/store.go:1075`. The commit-point rule cannot tell an operator's release from an internal clear. The new closing sentence ("a task is left with an empty status only by a release") holds by definition but hides this. I did not find another internal nil writer within my call budget.

No finding against F2, F3 or F5. F2's marker survives a hand pass and a detached first pass, and the `--continue-pass` bound now checks it. F3's probe full pass is refused in pre-run before `reopenWindow` (`battenPreRun`: `preflightRunOwner` runs before `reopenWindow`), so it opens no window on a stale verdict. F5's JSON payload matches the verb's help example (`internal/boardcli/cli.go:636-637`).

Calls:
- Read `check()` in `internal/battenshed/innerrun.go`, to see what runs under coverage today; my context held only the review's summary of it.
- Grepped for nil board-status writers, to test F6's release rule against internal clear paths; my context had not seen them.
