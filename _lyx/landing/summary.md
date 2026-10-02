# Agent names `<shortname>:<slug>:<role>` for every strand, with parentage

Every strand lyx spawns through reed now carries one hierarchical name: `<shortname>:<slug>:<role>` in a task worktree and `<shortname>:<role>` in the prime or a standalone run.
The one name is the reed strand name, the `LYX_STRAND_NAME` env value, the tmux pane title and the Claude session name.
A worktree run records the full name of the session that spawned it, and every strand in that worktree is told it as `LYX_PARENT`.
Escalations from a driver can then reach the spawner through Claude Code's SendMessage.

## What changes

- **Shortname.** Each repo has a shortname (`[a-z][a-z0-9]{1,5}`) recorded as `.lyx-shortname` on weft:main beside `.lyx-warp`.
  `lyx fabric clone --shortname <shortname>` gives it when a weft is first bound, and a later clone derives it.
  The new `lyx fabric shortname [<shortname>]` prints the record or records it for an existing hub; it refuses a different value once one is recorded.
- **`internal/agentname`.** A new leaf package is the sole former, parser and validator of names.
  `CONSTRAINTS.md` gains the Agent Name Invariant.
- **reed.** The name is formed in `AddStrand` under the state lock and stored as a birth attribute.
  A second strand of a role takes the lowest free `-N` from 2, counted against live and dormant strands; an explicit name already held refuses.
  Launch and resume inject `LYX_STRAND_NAME` and `LYX_PARENT`, and the Claude engine passes `--name "$LYX_STRAND_NAME"` through `internal/shell` (new `EnvRef`, bash and pwsh).
  Pane titles mirror the name with `allow-set-title off`.
  The watchdog repairs pane-title drift and, through a provider seam that `cliwire` fills with a Claude implementation, session-name drift; each repair is logged.
  `lyx reed list` is the hub-wide directory with a drift flag.
  `--name` and `--if-absent` accept the role segment or the full name.
- **Parentage.** The shed run seed gains a write-once `parent`, filled from the caller's `$LYX_STRAND_NAME` by `lyx batten run` and `lyx loom start`; batten copies it into the child's seed.
  `hubgeom.ReedGeometry` reads the default run's seed and tells reed.
- **Standalone runs.** `standalonegeom` derives the shortname as `s` plus the first five hex digits of the target's hash8; `LYX_PARENT` is never set there.
- **Drivers.** The ly-drive skill and the loom driver prompt send a short notice to `$LYX_PARENT` on escalation, in addition to the stop report.
- **Retired.** The `strand_name` key and its `<ROLE>`, `<ROUND>` and `<SHORT_GUID>` tokens leave reed's `Config` and both `reed.yaml` templates.
  An old `reed.yaml` still loads with the key ignored, and `lyx config reconcile` removes it.
- **Docs.** `docs/overview.md`, the reed and fabric package docs and the refusal spec are updated; nothing under `manifest/` changes.

## Design decisions to know

- A spawn on a hub with no recorded shortname refuses and names `lyx fabric shortname <shortname>`.
  A hub never falls back to a derived shortname; only standalone runs derive one.
- A task worktree whose name fails the slug grammar refuses a spawn with a way forward.
- Full names never become tmux session or socket names, since tmux rewrites `:` and `.`.
  They go only into pane titles, env and state; delivery and lookup key on strand state alone.
- `Strand.Parent` stays a layout guid; the naming parent is a separate field.
- A seed's `parent` is never rewritten by a later start, and a mismatch is not refused.
- There is no machine-wide check that two hubs share a shortname.

## To check by hand

- **Rollout order.** Right after `update-plugins.sh`, run `lyx fabric shortname ly` in loomyard's hub; until then every spawn on that hub refuses, including runs in flight.
- **Hand-started hub.** Until `lyx orch start` hosts the hub, the hub session records itself as parent with `LYX_STRAND_NAME=<shortname>:orch lyx batten run <slug>` and names its Claude session with `/rename`.
- **tmux.** Confirm the tmux build on Linux and on Windows accepts `allow-set-title`.
- **Claude internals.** The session-name repair reads the transcript's latest `custom-title` entry; if that format changes, the watchdog retypes `/rename` on each idle tick and logs each repair.
- **SendMessage.** Escalate once from a real driver to a live parent session to confirm delivery.
