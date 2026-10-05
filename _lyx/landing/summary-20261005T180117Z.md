# Roles get procedure from stencils; lyx types their skills

Every role lyx spawns now gets its whole procedure from a stencil, and lyx itself loads the skills the role needs.
No session depends on an agent remembering rules, reading file-memory or choosing to load a skill.
Every spawned role escalates to its parent instead of the operator in its own pane.

## What changes

- **Skills are typed by lyx.**
  `shuttleengine.Spec` gains `Skills` and `SkillLoadTimeout`.
  `claudeengine` types each skill as a submitted `/<name>` turn, in order, before the run's prompt.
  A skill that ends no turn within the timeout, or that the provider reports unknown, is logged and skipped; it never fails or hangs a launch.
  An empty list launches as before.
- **Per-role skill lists.**
  The module that owns a role constant declares that role's skills beside it.
  The "load the writing skills" steps are gone from the discussion, plan and rework stencils.
- **Parent directive.**
  The new `internal/parentdirective` package renders a shared directive into each spawned role's opening stencil.
  It names the parent as the escalation target, bans questions to the operator in the pane, and tells the role to act on a parent's message as on an operator instruction within its limits.
  A no-parent variant keeps the old behaviour of writing the report and stopping.
  The interactive discussion role omits only the operator-ban line.
- **Orch role stencil.**
  `contracts/stencils/orch/orch-template-role.md` carries the orch's whole procedure: the run loop, parent-review, escalations, PR-Gate, board policy, messaging and a table of what each role can do.
  The start, resume and adopt stencils keep only their path-specific line.
- **Orch note.**
  A note template replaces `scribe:handoff` in both handoff stencils, which are now one-line requests.
- **Cycle and distill.**
  `lyx orch cycle` always clears and the new `lyx orch distill` always compacts, whatever `cycle_mode` says; automatic triggers still follow `cycle_mode`.
  Both write the note first, and nothing is cleared or compacted without a passed note gate.
  A request whose note is not written within the handoff timeout is abandoned, and a stale marker left by a dead watcher is removed.
- **Reload.**
  The watcher types the skills, then a one-line pointer to the rendered role file and the note, after `/clear`, after a compaction it ran, after an auto-compaction, at `start` and at `--adopt`.
  Every step persists its phase before typing, and a restarted watcher re-types an unconfirmed step.
- **Driver stencil.**
  The loom driver launches from `contracts/stencils/shed/shed-template-driver.md`.
  The `ly-drive` skill is removed from the `ly` plugin, and the orchestrator-fork and operator-launched modes go with it.
  The recipe-blind and park-marker pins now run against the stencil.
- **PATTERN.**
  `PATTERN-role-skills-typed` and `PATTERN-parent-directive` are added, each backed by a scan over the registered stencils.

## Design decisions to check

- The slash-command form lives only in `claudeengine`; `shuttleengine` and `orchengine` carry skill names (`PATTERN-shuttle-provider-seam`).
- The skill-load timeout is one provider-neutral spec field, defaulted by `claudeengine`.
  The orch hands its handoff timeout to both its launch spec and the watcher.
- The role-to-skills table is a judgment call: every role that writes prose or code into the repo gets `scribe:prose`, and code-touching roles also get `scribe:code-quality` and `scribe:testing`.
  Language-specific scribe skills are deliberately left out.
- The parent name is one per worktree, taken from the same origin record as `LYX_PARENT`; a role spawned inside the run names that parent, not its direct spawner.
- `lyx orch cycle` changes meaning from "cycle in `cycle_mode`" to "clear".
- The orch stencil's run-loop section describes the current `setsid nohup` detach and must change when `lyx batten start` lands.

## Not covered

- Skills are not re-typed after Claude Code auto-compacts a spawned role other than the orch, since no watcher observes those sessions.
- Claude Code's reaction to an unknown `/<skill>` was not verified against a live session; the detection is exercised only against fakes.

## Check by hand

- Run `lyx orch start`, a `cycle`, a `distill` and `--adopt` against a real session and confirm the skills load, the role pointer is typed and the note is named.
- Launch a real run and confirm a spawned role asks its parent, not the operator, when blocked.
- Run `go test ./...` and the orch smoke suite.
- The deployed stencil copies on the board have drifted from this branch's sources; a production deploy by the operator refreshes them.
- After landing, delete the hub machine's file-memory entries the orch stencil absorbs: always-give-reed-attach, hub-acts-autonomously, proposals-on-board, bundle-tasks, hub-in-design, board-via-lyx, review-against-goal, bugs-to-gh-issues, triage-not-on-board, reviewer-forks-keep-context, message-strands-directly, hub-approves-pr-gate.
