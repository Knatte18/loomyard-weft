# lyx pause and resume: ordered hub-wide stop for safe suspend

`lyx pause [<slug>]` gives the operator one intentional stop that makes the hub safe to suspend within seconds, and `lyx resume [<slug>]` lifts it.
Without a slug the pause is hub-wide; with one it covers a single run's pair.
`lyx pause --status` lists the live entries and changes nothing.

The old bare boundary pause let an agent turn run on for up to an hour, and an unplanned sleep cut network calls and agent turns, then counted the slept time as idle.
Both are addressed here.

Closes #567

## What changes

- **Pause record.** The new leaf package `pauserecord` holds one JSON record of live and resumed entries, under a lock, at a path declared in `hubgeom` and `standalonegeom`.
  It survives a reboot, and a resumed entry is kept for 7 days so idle clocks can subtract the interval.
- **Pause sequence.** `pauseengine` (CLI in `pausecli`) writes the entry first, waits out a grace (`--grace`, 0 to 60 s, default 10), then interrupts each busy covered session with Esc through `Runner.Interrupt`, at most twice.
  It skips the caller's own strand and kills nothing.
  The report names interrupted, idle and refused sessions, network operations still in flight, parent-lock holders and waiters, and covered runs on Go producers.
- **Resume.** `lyx resume` moves the entry to the resumed list, then sends one fixed resume line through the guarded paste-submit to each session the entry owns.
  It skips dead, busy and draft-holding sessions and names each with its reason.
- **Holds under a covered run.** Shuttle run starts, network git in `fabricengine` and `gitexec`, and every `githubclient` call wait at entry.
  `lyx loom start`, `lyx loom resume`, `lyx batten run` and `lyx shuttle send` refuse and name `lyx resume`.
  `lyx reed resume` and batten's driver revive leave covered strands down.
  A held call prints one line naming the pause.
- **Readers treat a pause as neither idle nor failure.**
  Batten notices, shuttle's `Wait` notices and deadline, the shed loop watchdog, the parent lock's idle release and the orch watcher all exclude paused intervals.
  A covered run keeps its parent lock, and a covered waiter yields its queue turn to uncovered runs.
  Loom and batten status carry `paused_since`.
- **Orch.** Under a hub pause the orch watcher, the orch's only writer, interrupts a busy orch at the grace end, holds its notices and queues the resume line on resume.
- **Boundary pause retired.** A bare `lyx loom pause` and a bare `lyx shed pause` against a loom run refuse, naming `lyx pause <slug>` and `lyx loom pause --after <current_producer>`.
  `lyx webster pause`, webster's pause flag and `ErrPaused` are removed.
  `lyx batten pause` stays.
  `--after` naming a row that waits for the parent lock now ends that wait with the existing halt.
- **Unplanned sleep.** Sleep detection moved from batten into `pauserecord` and is shared.
  The shed loop watchdog and shuttle's deadline measure awake time, and batten's quiet notice after a sleep counts awake time only, which fixes the eleven-hour notice.
  An API-error stall across a detected sleep gets one automatic continue line per turn.
  Burler's live-fix resume delivers the task prompt when the strand holds only the skill-load preamble.
- **#567.** A watchdog kill names the watchdog, window, last activity and idle time in the halt note and trace.
  `run.json` now records the shed run directory and step that started a shuttle run, and `loomcli` stops the killed step's live shuttle runs, never the driver or other steps.

Docs, `PATTERN.md` (new `pauserecord` leaf row, orch-pane single writer now allows the hub-pause interrupt), the refusal spec, stencils and sandbox suites change with the code.

## Design decisions to review

- Go producers keep running under a pause; only agent sessions and network work are held.
- Agents may pause or resume only their own pair's slug; the check reads the caller's strand identity and guards against a mistake, not a determined agent.
- A session is recorded in the entry before its Esc, so a pause killed between the two can cost one surplus resume line, which its text tells the session to end on.
- A per-run entry lets through the network calls of a run that already holds the parent lock, so a held call cannot extend the lock's hold; the report names each.
- `lyx batten pause` is kept because its seam ends batten's wait mid-row, which the orch's hold-then-goto needs.

## To check by hand

- Run `lyx pause` and `lyx resume` against live runs and the orch, and confirm the interrupted sessions take up their turn.
- Suspend and wake the machine, with and without a pause, and confirm no idle notice, watchdog kill or deadline timeout for the slept time.
- A hub-wide pause left on silences every notice; `lyx pause --status` and `paused_since` are the only signals, and `runs-overview` shows a covered run as `running` unless it reads `paused_since`.
- The creel task will move `lyx shuttle send` and the orch watcher's typing behind creel; its delivery must honour pause coverage, which the creel entry does not yet say.
