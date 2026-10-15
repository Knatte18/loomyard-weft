# Combine the loom checklist, name seats, drop SingleLLMProducer

## What changes

**Checklist pane (`lyx loom checklist`).**
The pane is one combined frame: recipe rows in recipe order, each visited row's log lines (start, time, outcome, output, gate attempts) indented under its own box line, and a row not yet reached as one bare `[ ]` line.
A finished row no longer appears twice.
A review segment (a recipe segment that holds a Bouncer row) renders as one block grouped by round in run order: bouncer, review, fix, then the next round.
Arbiter attempts show in the round they decided, read from each attempt's envelope; an `awaiting` entry reads `escalated`.
A Stuck entry that took its row's `OnStuck` edge reads `bounced`; a halt still reads `stuck`.
A visit's time excludes paused time (`pauserecord`), so a row that took minutes no longer reads as hours across a pause.
`lyx batten overview` and `lyx loom status` are unchanged.

**Seat names.**
A lone seat carries no seat name: its strand role is the table's role prefix alone (`burler-review`, `burler-fix`, `discussion`, `plan`).
The first seat of a table is the lead; every later seat carries the name the recipe or config gives it, and its role is `SeatRole(prefix, name)`.
`seatengine.RoleChair`, `RoleAdvisor` and `AdvisorName` are gone.

**One LLM producer.**
`SingleLLMProducer`, `NewSingleLLMProducerGated`, `SpecSource`, the generic `SingleLLM` registry row and the `DiscussionSeats` row are removed.
Discussion-Write, Plan-Write, Darn-Write, Describe and PR-Rework now run on `MultiLLMProducer` over `internal/seatengine`, with a one-seat table where no advisors are configured.
`MultiLLMProducer` gained the one thing the single producer had alone: a fresh-spawn prompt amendment, which Plan-Write's rotation uses.

**Config.**
`discussion_producer` and `plan_producer` are retired in favor of two axes:

- `discussion_advisors` is a list of `{name, model}` maps (default empty).
- `discussion_lenses` and `plan_lenses` each name a `burler.yaml` fan whose lenses the lead forks (default empty, forks off).

All three are guard keys: only the hub-wide file or a worktree's own config sets them, never a local override or a task's `run_settings`.
`loomengine.MigrateConfig` rewrites old files on reconcile and reports each rewrite.

## Design decisions

- The lead of a multi-seat table is named by its row's role, not by the operator; the operator names only the later seats.
  This departs from the board entry's "the operator names each seat".
- A bounce stays `stuck` in the stored outcome; Shed writes the routing target to `routed_to` on the Stuck entry, and only the checklist maps that to `bounced`.
  `episodeStuckCount` still reads `routed_to` only on reroute entries, so the bounce budget is unchanged.
- The checklist maps an entry's output to its round through `shedadapters`, which owns the round file names.
- A machine sleep outside a pause, and a pause older than `pauserecord.Retention`, still count as row time.
- `chair` is no longer a shown name.
  A `discussion_advisors` entry migrated from a string list carries `advisor-<n>` until the operator renames it.
- Docs moved with the code: package `doc.go` files, `docs/overview.md`, `contracts/specs/shed-recipe-spec.md`, new `refusal-spec.md` rows, and `PATTERN-agent-name`.

## To check by hand

- No LLM bench of the zero-advisor table against the single writer ran before the single producer was removed, although the board entry asked for one.
  Field-by-field spec parity tests stand in for it; overrule if a bench is wanted first.
- `discussion-multi-llm-bench` cannot run its control arm (single against seats with no advisors) any more, and its pair configs must use `discussion_advisors` as `{name, model}` and `discussion_lenses`.
- `tools/tokencount` groups by role, so a report spanning this change shows both `burler-review-chair` and `burler-review`.
- A run in flight at deploy keeps showing `stuck` for bounces recorded before the change.
- A run started by `SingleLLMProducer` and live at deploy should be attached by the seat producer, not respawned; this was not exercised against a live run.
- Migration across layers: a hub default of `discussion_producer: seats` with advisors under a worktree file that sets no list of its own gets a report line asking the operator to set a named list.
  Check the report on the first reconcile after deploy, and that the hub's `discussion_advisors: ["sonnet[high]"]` does not start a dormant advisor.
- Unreadable arbiter attempts render as `arbiter attempt unreadable (<directory name>)` at the end of the frame, and an attempt with no envelope reads `running` before its `Deadline` and `no verdict` after; confirm both read sensibly in a live pane.
- A goto into a segment's review row carries no output, so it stands at the end of the round in progress; confirm this reads correctly in a run that circles.
- No production file should name `SingleLLMProducer`, `RoleChair` or `AdvisorName`: a repo-wide grep showed none, but re-run it after merge.
