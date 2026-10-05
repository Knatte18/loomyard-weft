# Stencils carry role procedure; lyx types skills, orch cycle/distill

Every role lyx spawns now gets its whole procedure from a stencil, and lyx itself types the skills the role needs.
No session depends on an agent remembering rules, reading file-memory or choosing to load a skill.
The board entry lists no issues, so this change closes none.

## What changes

- **Skills typed by lyx.**
  `shuttleengine.Spec` gains `Skills` and `SkillLoadTimeout`.
  `claudeengine` types each skill as a `/<name>` turn in order, then delivers the prompt pointer.
  The slash syntax lives only in `claudeengine`.
  A skill that ends no turn within the timeout, or that the pane reports unknown, is logged and skipped; it never fails or hangs a launch.
- **Skill list per role.**
  Each spawning module declares its role's skills beside the role constant.
  The "load the writing skills" steps are gone from the discussion, plan and rework stencils.
- **Parent directive.**
  The new `internal/parentdirective` package renders one directive from `contracts/stencils/parent/` into each spawned role's opening stencil.
  The role escalates to its parent by `SendMessage`, never to the operator in its pane, and acts on the parent's message like an operator instruction within its role's limits.
  The interactive discussion role omits only the operator-ban line.
  With no parent recorded, the no-parent variant keeps the old behaviour: write the report and stop.
- **Orch role stencil.**
  `orch-template-role.md` carries the orch's whole procedure: run loop, parent-review, escalations, PR-Gate, board policy, messaging and a table of what each role can do.
  The start, resume and adopt stencils keep only their path-specific line.
- **Orch note, `cycle` and `distill`.**
  `orch-template-note.md` replaces `scribe:handoff` in both handoff stencils.
  `lyx orch cycle` always clears and the new `lyx orch distill` always compacts, whatever `cycle_mode` says; automatic triggers still follow `cycle_mode`.
  Both write the note first, and a request whose note or idle probe does not pass within the handoff timeout is abandoned with nothing cleared or compacted.
- **Watcher reload.**
  After `/clear`, after a compaction it ran, after an auto-compaction, at `start` and at `--adopt`, the watcher types the orch's skills and then a one-line pointer to the re-rendered role stencil and the note.
  It types only when the idle probe passed on that tick, and persists each step so a restart re-types an unconfirmed step and never a persisted skip.
- **Driver stencil.**
  The `ly-drive` skill is removed from the `ly` plugin.
  Its procedure is now `shed-template-driver.md`, rendered as the loom driver's launch prompt.
  Only the loom-launched mode remains, and the stencil stays recipe-blind.
- **PATTERN.**
  `PATTERN-role-skills-typed` and `PATTERN-parent-directive` are added, each backed by a scan over the registered stencils.
  Docs (`doc.go` files, `docs/overview.md`, `README.md`, `plugins/ly/skills/INDEX.md`) are updated.

## Design decisions to check

- A skipped skill is visible only in the log.
  A spawned role then runs on its stencil alone; the orch also loses the board mechanics `ly:board` carries until the next reload.
- The parent name is one per worktree, taken from the origin record that reed injects as `LYX_PARENT`.
  A role spawned inside the run names that same parent, not its direct spawner.
- The directive widens trust only for the parent.
  A role still obeys every limit its stencil and settings set, and trusts no other session's message.
- The orch stencils are exempt from the ask-the-operator scan, since the orch answers to the operator.
- The orch stencil's run-loop section describes today's detached `lyx batten run`, and must change when `lyx batten start` lands.
- `lyx orch cycle` changes meaning from "cycle in `cycle_mode`" to "clear".

## To check by hand

- Claude Code's reaction to an unknown `/<skill>` in a real pane was not exercised against the live client; the skip is tested against the fake session and the pane-text detection.
  Type a nonexistent skill name into a spawn and confirm it is skipped within the timeout.
- A Webster master that Claude Code auto-compacts loses its skills, and nothing re-types them; only the orch is watched.
- The orch smoke suite and a real `lyx orch cycle` and `lyx orch distill` against a live orch were not run here.
- Boards copies of the stencils report drift until a production deploy; that is the operator's `update-plugins.sh` step after landing.
- After landing, the hub machine's file-memory entries that the orch stencil absorbs can be deleted.
