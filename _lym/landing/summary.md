# dobby: send a landing notice to the orch on request, per task

A run that lands sends nothing today, so the orch learns of a landing only when something else wakes it.
Dobby now posts one creel line to its worktree's nearest live orch when a run it drives lands or ends without landing.
The line reads `[dobby notice] <slug>: landed as <commit>`, `landed, commit unknown`, or `ended without landing`.
A beam counts as a run: the dobby that lands a beam into its parent after `lym beam ready` sends the line.

## Behavior

- New `beam.yaml` key `land_notice`, beside `max_runs`, with the values `all`, `per-task` (default) and `off`.
  `all` notices every run the dobby drives, `per-task` only a run whose board entry carried `notify_on_land`, `off` none.
  Any other value is refused, naming `lym config beam --set land_notice=<all|per-task|off>`.
  The value is read when a run is seen past Land, so a change applies to every run not yet noticed.
- New board field `notify_on_land`, set with `lym board upsert` and stored only when true.
  It is not scope: the run lock and the revised date ignore it, so it can be set on a running entry.
  A note carrying it is refused.
  Dobby clears it through `Board.ClearNotifyOnLand` once the line has gone out.
- The reed overview's dobby section and `lym beam status` show the `land_notice` value in the dobby header, and `notify_on_land` on each run or waiting task that carries it.

## Design decisions

- **One shared past-Land test.**
  `landFinished` in `internal/hibernateengine` is replaced by `battenrecipe.PastLand`, which holds the same rule, so `lym hibernate request --after` behaves as before.
  `battenrecipe.LandOutcomeOf` adds the outcome: empty while ahead of Land, `landed` when the history holds a Land Done, `ended without landing` otherwise.
  battencli computes it for each run listed to dobby, so `dobbyengine` imports no recipe package.
- **Dobby remembers the request.**
  Land prunes a landed task's board entry, so the field cannot be read afterwards.
  The dobby state record keeps a per-slug entry, created when a listed run is first seen ahead of Land and refreshed each tick while the board entry exists.
  A run first seen already past Land gets no line, so a fresh or restarted dobby does not notice old runs.
  The entry is dropped once the line went out, once the decision was no line, after three failed posts, or when the run is no longer listed.
  A failed clear of the field is logged and not retried.
- **Finalize records the landing commit.**
  `landingshed` gains a `RecordLanding(commit)` seam, called with the parent's head right after each successful parent-side merge, fresh or already landed, before the push.
  `loomcli` writes it to a new `shedrun` path under the run's scratch directory, and battencli reads it onto the run view.
  A failed record is logged at Warn and never changes Finalize's verdict.
  The commit is the parent's head after the merge, so it holds for a squash, a merge and a pull request merged on GitHub alike.
- **"Ended without landing"** means past Land with no Land Done, as a goto to Worktree-Teardown leaves a run.
  A cancelled run already notifies the orch through the cancel and gets no line.

## Known limits

- A field set after the run passed Land gets no line.
- A run that passes Land while the dobby is down long enough for a fresh state record to be written gets no line.
- The line needs a live orch; three failed posts drop it.
- Moving `lym hibernate request` into dobby is out of scope.

## To check by hand

Not run end to end against a live orch: set `notify_on_land` on a running task and confirm the line arrives and the field clears when the run lands.
