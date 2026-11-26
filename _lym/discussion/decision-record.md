# Decision record — the dobby watches every run it drives and mails its parent

## Goal

The parent orch hears of every stop of every run it parents, and of a gate slot a run holds too long, with no Monitor kept by hand.
The dobby of the run's parent worktree does the watching, so it needs no re-arming and survives the orch's context cycles.
It mails the parent on:

- every change of a run into a state other than running: halted (blocked, failed, awaiting), paused, the child's shed done, landed, cancelled;
- a gate slot held by the run past a bound derived from the tier's `timeout_min`, naming the holder's PID and its child processes;
- every row change, when the task's setting asks for progress.

## Scope

In:

- A run watch in `internal/dobbyengine`, run from the dobby's tick over the runs it drives (D1, D3, D4, D5).
- A per-task beam setting `beam.run_watch` (new) choosing `stops`, `rows` or `off` (D2).
- A gate-hold watch over `internal/gateslot`'s slot holders, with two hub keys in `beam.yaml` (D6).
- The orch role's Notices section and the dobby's doc and template naming the new lines and keys (D7).

Out:

- Any change to an existing notice (D8).
- Killing a hung gate: the watch reports, never kills; the tier timeout's tree kill is `dotnet-gate-fixes`'s (T4).
- Slots held outside a run the dobby drives, such as a hand `lym gate test` in the prime.
- Runs no dobby drives, such as a hand `lym loom start` outside a board task.

## Decisions

### D1 — The dobby watches each run's reading on every tick

On every tick that runs its steps, the dobby reads, for each run it drives, a watch reading of four parts:

- the run's derived state, `RunView.Run`, which `battencli.DobbyRuns` already fills through `runfsm.Derive`;
- batten's own state and row;
- the child's state and row, from the run notices' existing child reader, read only when batten's or the child's status file changed since the last reading;
  an absent child status, as after Worktree-Teardown, keeps the child part last read;
- an episode stamp: the history length of the layer that stands at a stop, batten's when batten's state is not running, else the child's.

The watch step runs before the done-run clear in the tick.
A change of the reading is what D2's level judges and D3's line reports.

Rationale: the orch's Monitor printed every change of state or row, and the reading holds every input that Monitor read through `lym loom status`.
Batten's own state is part of it because a batten halt outside Run-Shed, such as a pause seen during the Land lock wait, derives as running or landing, not halted, and today sends no notice.
The episode stamp tells a second stop at the same row from the first when the brief running reading between them fell between two ticks, as batten's notice keys tell episodes apart.

### D2 — `beam.run_watch` sets the level per task: `stops`, `rows` or `off`

`beam.yaml` gains `run_watch`, default `stops`, `CategoryNone`, read per task only through a new `dobbyengine.LookupRunWatch` (new) over `hubgeom.TaskSettingChain`, as `land_notice` is.

- `stops`: a line goes for each change into a stop (D3) of the derived state, batten's state, the child's state or the episode stamp; a row move alone, and a change back to running, send none.
- `rows`: a line goes for every change of the reading, including a row move and a resume.
- `off`: no watch line goes; the gate-hold notice of D6 is not governed by this setting.

`run_watch` joins the per-task keys `internal/beamsetting/beamsetting.go` declares and the per-task check in `internal/dobbyengine/config.go`, so a value outside the three is refused where a bad `land_notice` is, with the key's full name and its three values as the way forward.
A failed per-task read keeps the run's last level, as the landing notice keeps its last reading.

Bound: the setting only adds mail to the parent; it changes no gate, row or act rule.
`pattern/PATTERN-config-layers.md`'s per-task beam setting rule names `beam.run_watch` beside the three keys it lists.

Rationale: the entry asks that lym watch every run unasked, so the default is on; `rows` is the entry's optional progress level, per task through `run_settings` like the other beam settings.

### D3 — A stop is any reading whose state is not running

A reading is a stop when any of these holds: the derived state is paused, halted, cancelling, cancelled or landed;
batten's state is not running;
the child's state is not running.
The child's shed done, before batten lands, is a stop.
A landed or cancelled reading of a run whose landing-notice memory in the dobby's state record holds it as requested sends no watch line, since the landing notice reports it.

The line reads `[run watch] <slug>: <derived state>; batten <state> at <row>; child <state> at <row>`, then the child's or batten's reason when one is set, cut to one line, and `since <RFC 3339>`.
Paths under the task worktree render relative to it, as the run notices render them.

Rationale: the entry names blocked, failed, paused and done "whatever the cause"; reading every layer covers a halt whatever layer records it.
The skip reads the landing notice's own memory, so the two never disagree on whether that line goes.

### D4 — The dobby's state record keeps the watch memory and the owed lines per run

Each run's entry in the dobby's state record gains the last reading the watch handled and the run's owed watch lines, in order.
A reading the level or D3's skip sends no line for still becomes the memory.
A reading equal to the memory sends nothing, so a dobby re-exec or restart repeats no line.
A run with no memory, such as one first seen after an upgrade, takes its current reading as the change: a run standing at a stop then sends one line, so an upgrade sends one line per such run at once.
The done-run clear keeps a run's entry while it owes a watch line, as it keeps one that owes a landing notice.

Rationale: the run notices keep their memory the same way, and a stop told before a restart must not be told again, while a landed line must outlive its run's clear.

### D5 — Watch lines go untagged and in order to the nearest live orch, stamped with the run's autonomy

A watch line posts through `orchcli.NotifyRunFrom` as `<shortname>:DOBBY` or `<shortname>:<beam>:DOBBY`, stamped by `orchcli.WithRunAutonomy`, with no topic and no run tag.
So no watch line replaces another unread one, and a run's teardown drops none, the landed line included.
Each tick posts a run's owed lines oldest first and stops at the first that does not go.
While no orch strand is recorded, a line waits and counts no attempt.
A post that fails is tried again no sooner than a probe interval later, and dropped with a Warn after three failed attempts.
A line owed for an hour of awake time is dropped with a Warn, so an absent orch never holds a run's clear for long.

Rationale: the orch wants every change, in order, and a landed line posted just before teardown must survive it.
Holding a line while no orch strand exists keeps it across an orch restart, as batten's notice holds.

### D6 — The dobby notices a gate slot a run holds past its bound

On each probe tick the dobby reads the hub's slot holders through `gateslot.Pool.Holders`, which drops a record whose lock is not held.
A holder belongs to a run the dobby drives when the holder's worktree is that run's task worktree or a path inside it, compared on whole path segments.
Each run's task worktree is resolved from its slug as the dobby's file watch resolves it, independent of D1's child read.
Its bound is `gate_hold_factor` times its tier's `timeout_min`; a holder with no tier, or a tier with no positive timeout, takes `gate_hold_min` minutes.
`beam.yaml` gains both hub keys: `gate_hold_factor: 1.5` and `gate_hold_min: 90`; `gate_hold_factor: 0` sends no gate-hold notice.
A holder is past its bound when the awake time since its `Started`, through `pauserecord.AwakeSince` over the dobby's recorded sleeps, reaches the bound.
A failed load of the gate config skips the check for that tick with one Warn per failure episode.

A tier run records its environment marker, the entry every process it starts carries, in its slot holder record as a new field.
The notice is one `[dobby notice]` line about the run, under its own topic, gate-hold plus the slug, and the run tag.
It names the slot, tier, site, start and bound, the holder's process tree from `proc.TreeSample`, and every other process carrying the holder's marker through `proc.PIDsWithEnv`, each PID with its executable and CPU time.
It goes once per hold, keyed by slot, PID and start, and the key is kept in the run's entry of the state record; a new hold of the same run is noticed again.

Bound: the watch reads `/proc` and the slot records; it kills nothing, frees no slot and spawns nothing.
`Pool.Holders` is the reader `lym gate status` already uses; each dobby calls it at most once per probe tick.
A suspend the dobby did not record, such as one ending before it started, counts as awake time and can bring the notice early.
A holder with no marker, such as a non-tier holder, is named by its tree alone.
Both keys are `CategoryNone`: they bound only a notice.

Rationale: the hung tier3 gate of #216 held its slot 21 hours while the run read running, and the quiet notice stays silent while a verify wait is live.
#216's likely cause is a timeout that killed only `sh`, leaving an orphaned testhost outside the holder's tree; the marker names it.
The tier timeout's own clock stops while the machine is suspended: Go's runtime timers run on Linux's `CLOCK_MONOTONIC`, which clock_gettime(2) states excludes suspended time.
Measuring awake time keeps the bound on the same clock.

### D7 — The orch role and the dobby's docs name the new lines and keys

`contracts/orchrole/orch-role.md`'s Notices section names `[run watch]` lines as notices: a stop line is a stop, and a later `[batten notice]` or driver line for the same stop adds its detail and opens no second fork.
It names the gate-hold condition among the `[dobby notice]` conditions.
`internal/dobbyengine/doc.go` and `internal/dobbyengine/template.yaml` describe the watch, `run_watch`, `gate_hold_factor` and `gate_hold_min`.

Rationale: the orch reads a notice by its prefix, so a prefix it does not know is mail it may not act on.

### D8 — Existing notices stay as they are

The watch adds lines and removes none: batten's state-changed and driver-dead notices, the driver's hand-back lines, the dobby's run notices, `notify_parent_on` and the landing notice keep their behaviour and their holds.

Rationale: batten's notice waits for the stop report and for an arbiter or tackler in flight, so it carries the detail the orch acts on; the watch line is the prompt floor under it.

## Constraints

- `PATTERN-sole-parsers`: slot records are read only through `gateslot`'s pool.
- `PATTERN-daemon-wakeups`: the watch adds no loop and no timer; it runs inside the dobby's existing tick, and the gate check only on probe ticks.
- `PATTERN-owner-machine`: the dobby runs on the owner machine, where the run's slots and processes are.
- `PATTERN-agent-addressing`: every line goes through creel as the dobby's program name.
- `PATTERN-tolerant-records`: the state record's and the slot holder record's new fields rewrite through their existing writers, and an older record without them reads as no memory and no marker.

## Auto-mode assumptions

- The `stops` default of D2, so every hub gets the watch without setup.
- The `[run watch]` prefix of D3, so the orch tells watch lines from batten's and the dobby's run notices.
- D6's bounds: the entry's factor 1.5 and a 90-minute fallback.

## Open risks

- Mail volume: each stop now reaches the orch once more, as the watch line ahead of batten's notice.
- `beam-member-parent` (T12): until it lands, the nearest live orch a beam member's dobby notice resolves to can be the prime's orch rather than the beam's, and watch and gate-hold lines go the same way.

## Acceptance criteria

- A driven run whose child goes from running to blocked at a row posts one `[run watch]` line naming the derived state, batten's and the child's state and row and the reason; a later tick with the same reading posts nothing (D1, D3).
- A child that blocks at a row, resumes and blocks at the same row again with no tick between posts a second line, through the episode stamp (D1, D4).
- A batten blocked outside Run-Shed while its child reads done posts a stop line (D1, D3).
- Under `stops`, a row move while running, a resume to running and batten's row moves after the child's done post nothing; under `rows` each posts one line; under `off` no reading posts and a gate-hold notice still goes (D2, D6).
- `run_watch` resolves through the per-task chain: the task's own worktree value, else a parent worktree's, else the hub default, else `stops`; a value outside the three is refused by the per-task check with the key and its values (D2).
- A dobby restarted with the state record's memory posts no line for a reading already told; one with no memory posts one line for a run standing at a stop (D4).
- A landed run posts one watch line when its landing-notice memory reads not requested and none when it reads requested; the child status vanishing at teardown posts no extra line (D1, D3).
- A landed run whose watch line has not posted is not cleared until the line posts or is dropped (D4, D5).
- Owed lines post oldest first; with no orch strand recorded they wait without counting attempts and post once one is; a failing post is retried no sooner than a probe interval and dropped after three failures; a line owed an hour of awake time is dropped (D5).
- A posted watch line carries no topic or run tag and the run's autonomy stamp (D5).
- A slot held under a driven run's task worktree by a tier with `timeout_min: 35` and factor 1.5 posts no notice at 52 minutes of awake time and one at 53, naming the slot, tier, PID, each child PID and a process outside the tree that carries the holder's marker; a sleep interval inside the hold delays it (D6).
- The same hold posts once, also across a dobby restart; a new hold with another PID or start posts again; a holder in a sibling worktree whose path only shares a prefix posts nothing; a holder with no tier takes `gate_hold_min`; factor 0 posts nothing; a failed gate config load posts nothing and warns (D6).
- A gate-hold notice and a later run notice of the same run both reach the orch unread, neither replacing the other (D6).
- Existing notice tests in `internal/battenshed`, `internal/dobbyengine` and `internal/loomcli` pass unchanged (D8).
- Review: `contracts/orchrole/orch-role.md`'s Notices section names `[run watch]` lines and the gate-hold condition as D7 says (D7).
- Review: `internal/dobbyengine/template.yaml` comments `run_watch`, `gate_hold_factor` and `gate_hold_min`, and `internal/dobbyengine/doc.go` describes the watch (D2, D6, D7).
- Review: `pattern/PATTERN-config-layers.md`'s per-task beam setting rule names `beam.run_watch` (D2).

## Notes for the plan writer

- The child reader the run notices use, `RunNoticeDeps.ReadChild`, already returns the child's state, row, reason and pair directory.
- `internal/dobbyengine/landnotice.go` shows the per-task lookup and a per-run memory in the state record; `internal/dobbyengine/donerun.go` holds the done-run clear D4 extends.
- `gateslot`'s config loader is strict, so the dobby wiring in `internal/beamcli` loads the gate config.
- `internal/gatecli/processcap.go` forms the tier run's environment marker.
- If `config-keys-nested` has landed by Plan time, the refusal names `beam.run_watch` through its key-name helper.
