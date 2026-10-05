# ly-drive as a stencil, a ly:runs skill, and lyx-injected skill loading

`lyx-skills-and-stencils` · Task · enhancement, stencil, orch, shed

Depends on: [`orch-cycle-hardening`](design-orch-cycle-hardening.md), [`loom-bouncer-rounds`](design-loom-bouncer-rounds.md)

# Goal

Every agent can drive lyx without rediscovering its workflows through `--help`, every role lyx spawns gets its procedure from a stencil, and no lyx session depends on an agent choosing to load a skill.

# Decisions (operator, 2026-10-05)

- **Skill or stencil.** A skill carries what any session needs to use lyx, the operator's own sessions and the orch included.
  A stencil carries one spawned role's procedure.
- **Skills are workflow guides, not references.** Cobra's `--help` stays the reference for flags and payload keys; a skill gives the concepts, the order of steps and one example per verb, like `ly:board`.
- **lyx injects skill loading; it never relies on a hook.** The scribe plugin's `SessionStart` hook asks a session to load `scribe:conversation` and `scribe:prose`, and on 2026-10-05 the orch saw the request and did not load either.
  A handoff skill would have the same weakness.

## The ly-drive skill becomes a stencil prompt

The `ly:ly-drive` skill exists only to start the drivers that run a loom through Shed with repeated `lyx shed step`.
Every other Claude process lyx spawns gets its instructions from a stencil prompt, and the driver should too.

- Move ly-drive's content into a stencil under `contracts/stencils/` and have the launch prompt render it.
- Remove the `ly-drive` skill from the `ly` plugin.
- `loom-bouncer-rounds` edits `plugins/ly/skills/ly-drive/SKILL.md` (its D6, the `parent_notice` to the parent); carry that edit into the stencil.

## A `ly:runs` skill for the run loop

The hub orch on 2026-10-05 had to find the loop through `--help`: what a skill should have told it.

- Spawn a task with `lyx batten run <slug>` from the prime, in the background; read progress with `lyx batten status`, and use `pause` and `goto`.
- A parent-review request arrives as a message naming a brief; a one-shot fork reads it, reviews and submits with the brief's `lyx loom review` command.
- Escalations from a child (CIRCLING, a spent review budget) arrive the same way, once `loom-bouncer-rounds` has landed.
- Landing and what the hub does after it, such as amending board entries a landed task superseded.
- Every lyx agent is reachable by name through Claude Code's `ListAgents` and `SendMessage`.
  Names follow `internal/agentname`'s fixed form: `<shortname>:<role>` in the prime, such as `ly:orch`, and `<shortname>:<slug>:<role>` in a task worktree, such as `ly:<slug>:driver` or `ly:<slug>:discussion`; reed injects a strand's own name as `LYX_STRAND_NAME` and its parent's as `LYX_PARENT`.
  So the orch can address any run's driver or writer directly, for a scope change or a resume, without going through the operator.
  Only `internal/agentname`'s doc states the form today; no skill or stencil tells an agent.
- Same shape as `ly:board`: concepts, steps, one example per verb, `--help` for the key set.

## The orch watcher injects skill loading at every entry

The orch prompt opens with an imperative to load `scribe:conversation`, `scribe:prose`, `ly:board` and `ly:runs` before anything else.
It reaches the session on every path in: `lyx orch start`, `--adopt`, and the resume after a cycle.
After a compact cycle the watcher types that prompt once the compaction has finished, since `/compact`'s focus argument only shapes the summary and the compaction drops the loaded skills' content.
`orch-cycle-hardening` rebuilds the cycle and the orch stencils; this builds on its result.

Settle which spawned roles also need the scribe skills: every role that writes prose for the operator or into the repo.
