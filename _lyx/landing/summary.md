# Add the Go driver per task, with the tackler behind it

A loom run can now be driven by Go instead of an LLM driver.
Go steps the run and settles the mechanical stops by a closed rule table.
A cold, cheap agent, the tackler, is spawned only at a stop no rule settles, and the arbiter only when Go's rule or the tackler's verdict calls for it.
Anything needing a parent choice still goes to the orch.
The LLM driver is unchanged and stays the default.

The reason: in the week of runs counted on 2026-10-12, no driver ever edited a file or fixed git, about half the stops were mechanical re-steps, and drivers sat idle 93 % of their life.

## What changes

- **Per-task choice.** A new loom config key `driver_kind` (`llm` or `go`, default `llm`) is open to the local layer, so `run_settings` sets it per task.
  Precedence at seeding: a typed `--child-driver` on `lyx batten run`, then `driver_kind`, then `llm`.
  The recorded driver never changes after seeding.
  Batten's prime seed now records `child_driver` only when typed; a typed flag against an adopted seed that records none is refused with its way forward and a refusal-spec row.
- **`lyx loom drive`.** A hidden role verb, the Go driver, spawned detached by the go arm of `lyx loom start` in place of `lyx loom run`.
  It calls the `shedverbs` `--until-stop` and goto bodies and restates neither.
  It holds a driver lock of its own for its whole life, settle waits included, while the run lock stays free between steps.
  Start, resume and batten read a held driver lock as a live driver.
- **Rule table.** Each stop takes the first matching row: done, busy, ownership, paused, arbiter-eligible awaiting, `parent_notice` relay, planned hand-off, verify-failure goto, binary hand-over, returning stop key to the arbiter, transient or `producer`/`bootstrap`/`unseeded` error to the tackler, `interrupted` by policy.
  The repair cap is two re-steps per stop key without the run advancing.
- **Tackler.** A cold, single-stop, read-only agent spawned through shuttle with a new recipe-blind stencil.
  Its verdict is `resume`, `hand-back` or `escalate`; Go validates it, and every malformed case reads as a `hand-back` with a named cause.
  Go bounds it after it ends: a changed stop key is a moved run, handled as a new stop; with the stop key unchanged, a changed HEAD, branch or uncommitted content reads as `hand-back` with cause `tampered`.
  New loom keys: `tackler` (default `haiku`), `tackler_timeout_min` (default 10) and `tackler_max_spawns` (default 4, 0 turns it off).
- **Arbiter.** The Go driver calls the existing arbiter spawn path in-process, so eligibility, `arbiter_max_spawns`, attempt records and idempotence are shared.
  On `escalate` the arbiter's stencil receives the tackler's case brief through a new marker, with lyx's own text to check every fact and never take the hypothesis as given.
- **Verify-failure goto.** The only goto Go performs by rule is to the target of a matching verify failure record, once per unwatched run.
  A batten-watched run's halt is left to batten.
  The one retry is counted from both sides: Go's settle record, which batten's `publishRetryDecision` reads, and batten's publish-retry marker, which Go reads.
- **Binary change.** Go compares the last step's recorded build against the on-disk `lyx` build, and with a clean tree hands over to a fresh `drive`.
  After a hand-back, batten's `InnerRun` wait re-steps a halted, driverless go child once per halt episode and build.
- **Notifications and records.** Parent messages go through `orchcli.NotifyPrime` by the LLM driver's two rules (watched and unwatched).
  Every hand-back writes a stop report, the park marker and settle records in the friction directory.
- **Resume.** `lyx loom resume` over a halted, driverless, go-seeded run now spawns a Go driver instead of refusing toward `lyx loom start`; its other branches are unchanged.
- **Moved runs and pauses.** The driver re-reads status before acting and treats a run moved by another actor as a new stop; a pause covering the slug holds its spawns and re-steps.
- **Empty-reason `paused` stop.** `shedengine` now sets a reason on the bare `pause_requested` pause and on a cancelled context, in both `stepLocked` branches.
  Go writes a friction note only for a `paused` stop whose reason is still empty.
- **Orch stencil.** One line says a go-driven run has no driver to message and is reached through `lyx loom` and `lyx shed` verbs.
  This departs from `orch-split`'s routing for go-driven runs only, until a later task adds a receiver.
- **Docs and patterns.** `internal/loomcli/doc.go`, `internal/battenshed/doc.go`, `internal/shedrun/doc.go`, `docs/overview.md`, `PATTERN-driver-choice-single-site` and the refusal spec follow.

## Design decisions to know

- `judge-cap-call` lands first; its changes to the arbiter stencil, loom config, loomcli wiring, runsettings tests, stencils registry and refusal spec are merged in here.
- Until `arbiter-when` and `judge-cap-call` narrow the list, the Go driver spawns the arbiter at every Bouncer escalation the LLM driver does, soft-cap escalations included.
- A tackler or arbiter timeout, or a malformed verdict, hands back to the parent and does not escalate.
- Both the Go driver's hand-over and batten's build check use the `lyx` executable resolved once at wiring, so they survive a deploy that replaces the binary.
- A standalone go-driven run with no batten has no binary watch after a hand-back.
- With a hub default of `driver_kind: go`, every new task switches drivers at once; the setting is meant per task first.
- A long-lived `drive` process holds its executable on Windows, as batten and the hub daemons already do.
- A live tackler strand meets `lyx pause`'s Esc and `lyx resume`'s typed line like any agent, which `pause-esc-rewind` reports can open the rewind picker.

## To check by hand

Nothing here ran against a real LLM; the tests fake the shuttle, clock and process seams.

- Run a real task with `run_settings` `loom.driver_kind: go` end to end, and watch one halt that reaches the tackler and, if possible, one `escalate` to the arbiter.
- Review findings left open, each to confirm is closed:
  - the binary re-step marker in `internal/battenshed/innerrun_binaryrestep.go` blocking the publish retry;
  - the `friction: failed` done notification in `internal/loomcli/drive.go` naming a stop report that is not written at that exit;
  - the second seed-driver reader in `internal/loomcli/resume.go` and the matching permitted-consumer list in `pattern/PATTERN-driver-choice-single-site.md`;
  - `internal/battencli/wire.go`: the build check re-resolving the `lyx` executable at each call;
  - `internal/loomcli/drivewiring.go`: the hand-over successor spawn re-resolving the executable.
- The fix commits for the last two make both use the executable resolved at wiring; confirm they hold across a deploy.
- A bare `pause_requested` pause and a cancellation return a `paused` step stop with a reason in `internal/shedengine/run.go`.
