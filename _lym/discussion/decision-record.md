# Decision record — landing-watch

## Goal

A dobby posts one short creel line to its worktree's nearest live orch when a run it drives lands or ends without landing, naming the slug, the outcome and the landing commit.
Which runs get the line is decided by one beam.yaml key, and a task asks for it through a board field, so the orch can keep a promise to report a run's result as soon as it lands.

## Scope

- One shared test for a batten run past its Land row, taken out of `internal/hibernateengine/watch.go`, which hibernate and dobby both call; it also tells a run that landed from one that ended without landing.
- A new beam.yaml key, `land_notice`, with the values `all`, `per-task` (the default) and `off`.
- A new board field, `notify_on_land`, set with `lym board upsert` and cleared by dobby once its notice went out.
- The landing commit: Finalize records the parent's head after its parent-side merge where the run's dobby can read it.
- The dobby step that sends the line, and its memory in the dobby's state record.
- The reed overview's dobby section, and `lym beam status`, which renders the same lines, show the key's value and which runs and waiting tasks carry the field.
- Out of scope: moving `lym hibernate request` into dobby (board note `hibernate-in-dobby`).

## Decisions

### D1 — The shared past-Land test lives in battenrecipe

`battenrecipe` gets `PastLand(state, producer)`, the rule hibernate's `landFinished` held (done, or at Worktree-Teardown in any state), and `LandOutcomeOf(status)`, which returns empty while the run is not past Land, `landed` when it is past Land and its history holds a Land Done, and `ended without landing` otherwise.
Hibernate's trigger calls `PastLand` in place of `landFinished`, which is removed, so its behavior is unchanged.
battencli's run listing for dobby calls `LandOutcomeOf` and hands the outcome to dobbyengine on each RunView, so dobbyengine imports no recipe package.
Rationale: `battenrecipe` already owns the batten row names both tests read, and hibernateengine already imports it.

### D2 — beam.yaml `land_notice`

A per-worktree key beside `max_runs`, category none, values `all`, `per-task` and `off`, default `per-task`; LoadConfig refuses any other value naming `lym config beam --set land_notice=<all|per-task|off>`.
`all` sends a line for every run the dobby drives, `per-task` only for a run whose board entry carried `notify_on_land`, `off` sends none whatever the field says.
The value is read at the moment a run is seen past Land, so a change takes effect for every run not yet noticed.

### D3 — Board field `notify_on_land`

A bool on the entry, stored only when true, accepted by every upsert path and carried by board replay as a field.
It is not scope: the run lock and the revised date ignore it, so it can be set on a running entry.
A note carrying it is refused, naming the way forward to clear it or promote the note.
`Board.ClearNotifyOnLand(slug)` clears it, and writes nothing when the entry is gone or does not carry it.
Bound: any board writer can set the field on a run-held entry; it changes only whether a creel line goes out.

### D4 — Dobby remembers the request while the run is ahead of Land

Land prunes a landed task's board entry, so the field cannot be read after the landing.
The dobby's state record gets a per-slug land-notice entry, created when a listed run is first seen ahead of Land, and holding whether its board entry carries the field, refreshed each tick while the entry is on the board.
A run first seen already past Land gets no entry and so no line, which keeps a fresh or restarted dobby from noticing old runs.
The entry leaves the record once the line went out, once its decision was no line, after three failed posts, or when the run is no longer listed.
Bound: a field set after the run passed Land, or a run that passed Land between a dobby's death and a fresh state record, gets no line.

### D5 — The line and the field's clearing

The line reads `[dobby notice] <slug>: landed as <commit>` or `[dobby notice] <slug>: ended without landing`, `landed, commit unknown` when no commit was recorded, through the existing Notice seam to the nearest live orch.
A beam's batten run is driven by its parent's dobby, so a beam's landing gets its line from that dobby with no extra path.
Once a line went out for a run whose entry carried the field, dobby clears the field through a new ClearNotifyOnLand seam; a failed clear is logged and not retried.
The step runs on every tick with no anchor change standing, after the passes, so it reads the full run listing before done runs drop out of it.

### D6 — Finalize records the landing commit

landingshed's Finalize gets a `RecordLanding(commit)` seam, called with the parent pair's head right after each successful parent-side merge, a fresh one or the already-landed path, before the push.
loomcli wires it to write the commit into a file under the batten run's scratch directory in the prime, named by a new `shedrun` path function; battencli's run listing reads it onto the RunView.
A failed record is logged at Warn and never changes Finalize's verdict.
Bound: a Finalize re-entered after a later landing on the same parent records that parent's newer head.

### D7 — The overview shows the key and the field

The dobby View carries the `land_notice` value, and each run and waiting task whether it carries `notify_on_land`.
`DobbyLines` adds the value to the dobby's header and `notify_on_land` to the line of each run or waiting task that carries it.

## Constraints

- PATTERN-shed-driver: the new run path is a `shedrun` function.
- PATTERN-durable-vs-ephemeral-state: the landing-commit file sits under the run's ephemeral scratch directory.
- PATTERN-tolerant-records: the state record gains keys through `internal/state`, as the rest of dobby's record.
- PATTERN-cwd-resolution: dobbyengine stays on told geometry and imports no lymcwd.
- PATTERN-config-layers: the key is read only through beam.yaml's layered load.
- PATTERN-agent-messaging: the line goes through creel via the existing Notice seam.
- PATTERN-test-speed: tests by `lym gate test`, tier0 where they fit.

## Auto-mode assumptions

- "Ends without landing" means past Land with no Land Done in its history, as a goto to Worktree-Teardown leaves it; a cancelled run, whose records the cancel removes, already notifies the orch through the cancel and gets no line.
- The field name is `notify_on_land`, as the entry suggests.
- The landing commit is the parent's head right after the parent-side merge, which is the landing commit for a squash, a merge and a pull request merged on GitHub alike.

## Open risks

- A Finalize that lands while the dobby of the parent is down for long enough that a fresh state record is written loses the line.
- The notice depends on a live orch; three failed posts drop it.

## Acceptance criteria

- `PastLand` holds exactly where hibernate's old test held, and hibernate's after trigger fires as before (D1).
- `LandOutcomeOf` tells a landed run, one that ended without landing and one ahead of Land apart (D1).
- beam.yaml loads `land_notice` with its default, takes each of the three values and refuses any other naming the way forward (D2).
- An upsert sets and clears `notify_on_land`; a run-held entry takes the change; a note carrying it is refused; ClearNotifyOnLand clears it and writes nothing when absent (D3).
- A run seen ahead of Land and then past it gets one line under `all`, under `per-task` only when its entry carried the field even after the entry was pruned, and none under `off`; a run first seen past Land gets none (D4, D5).
- The line names the slug, the outcome and the commit, and the field is cleared once a line for an entry carrying it went out (D5).
- A failed post is retried on the next tick up to three attempts (D4).
- Finalize records the parent's head after a successful parent-side merge and records nothing when the merge fails (D6).
- The overview's dobby header shows the `land_notice` value, and a run or waiting task carrying the field shows `notify_on_land` (D7).
