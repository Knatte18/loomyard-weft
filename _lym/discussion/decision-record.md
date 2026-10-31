# Decision record — dobby-start-failures

## Goal

A released task that dobby cannot start shows why in `lym beam status`, the orch hears of it within a pass or two, and a slug whose old run record is finished is held with a way forward instead of failing every pass.

## Scope

In scope:

- The dobby start step in `internal/dobbyengine` and the task reading it is fed from `internal/battencli`.
- The dobby lines `lym beam status` and the beam window print for a waiting task.
- The answers to the board entry's two "find out" points, given in the change description.

Out of scope:

- Refusing a board entry whose slug matches a finished run record when the entry is created or promoted; the board reads no run records.
- Changing `lym batten cleanup` or the done-slug refusal text itself.
- Creel delivery; creel did not hold the notice (see D4).

## Decisions

### D1 — A finished run record holds the task before any pass is spawned

`battencli.DobbyTasks` reads, for each released task, the status file of the slug's run record under the prime.
When that status reads `done`, the task carries the refusal a batten run of the slug would give, the same done-slug text with its way forward: `lym batten cleanup --apply` from the prime, the task branch removed, or a new slug.
Dobby's start step treats that refusal as a hold, like a pending dependency: it spawns no pass while the record stands, and starts the task on the first tick after the record is gone, with no backoff left to wait out.
Rationale: the pass fails before it can claim anything, and the existing record of its status file made dobby read every failed first pass as a claimed run, which cleared the backoff and spawned a new pass every tick.
Bound: the hold reads only the `done` state, the one state batten refuses as finished; any other state is a run batten resumes and is started as today.

### D2 — The hold notices the orch once per text, on the tick it is first seen

On the first tick that sees the hold, dobby posts the same `dobby could not start the task` notice a refused start posts, and records the text as noticed in the task's start record, so a later tick does not resend it.
A changed board entry resets the record as for a refused start, so the notice goes out again for the new entry.
Rationale: the board entry asks for a notice within a few passes and no repeat for the same error; the hold is known on the first tick, so the notice goes then.

### D3 — `lym beam status` names the reason for every task it does not start

A task held by a finished run record renders `waits on a start batten refuses: <text>`.
A task with nothing holding it back but a refused start whose backoff has run out renders `ready to start, last refused start: <text>`, so `ready to start` is shown without a reason only for a task that has not been refused.
Rationale: the incident showed `ready to start` for an hour with no reason; the refusal is in dobby's record and the view only has to show it.

### D4 — The causes the board entry asks for are answered in the change description, not in code

The late notice: a done run is dropped from dobby's run list every tick and its state entry with it, so the failing-pass count of the misread first pass never reached the notice threshold; the only notice came when `lym batten cleanup --apply` removed the record while a pass was in flight, so that pass read as a refused start.
Creel did not hold it.
The record of another machine: the board history shows an earlier task with the same slug ran from Neve and landed on 2026-10-28, and a new entry was filed under the same slug the next day; the run record is tracked under `_lym/` and reached the other machine with the prime.
Rationale: D1 removes the path that misread the record, so no code change is needed for the late notice beyond it.

## Constraints

- `PATTERN-dobby-starts-runs`: dobby still starts runs only from an empty status, in batten passes.
- `PATTERN-tolerant-records`: the dobby state record keeps its fields; no new field is needed, and none is refused.
- `PATTERN-refusal-way-forward`: the hold text names the way forward, and a test reaches the hold.
- `PATTERN-test-speed`: tests are table rows in the existing tier0 tests of the start step, the view and the dobby lines.

## Auto-mode assumptions

- The done-slug refusal text from `battencli` is the hold text, so the notice and the status show the same words a hand `lym batten run` prints.
- Reading one status file per released task each tick is cheap enough; dobby already reads the board and every run's status each tick.

## Open risks

- A run record that turns `done` while its first pass is in flight is still read as claimed; that is a run that finished, which is correct.
- A status file that does not decode is not a hold; the pass then refuses with the decode error, which the existing refused-start path backs off and notices.

## Acceptance criteria

- D1: a released task whose slug has a `done` run record is not started, and dobby spawns no pass for it; once the record is gone it is started on the next tick with no backoff wait.
- D1: `battencli.DobbyTasks` fills the refusal for a released task with a `done` run record and leaves it empty for one with no record or a record in another state.
- D2: the hold posts one notice on the tick it is first seen, and no second notice on later ticks while the record stands; a changed entry posts it again.
- D3: the dobby lines render a held task as `waits on a start batten refuses: <text>`, and a ready task with an expired refused start as `ready to start, last refused start: <text>`.
- D4: the change description states the cause of the late notice and the origin of the other machine's run record.
