## Goal

```markdown
## Seen (orch, 2026-11-16)

The bench `claude-code-mode` found no gain from Claude Code's Code Mode on the Plan writer,
and the web study `code-mode-web-research` (report `docs/research/claude/code-mode-web-research-2026-11-16/report.md`) found nothing that would change that:
Anthropic and users agree Code Mode pays on mechanical, read-heavy work, such as one change repeated across many files,
and not where each result needs a judgment, as in planning.

## Idea

A bench of Code Mode on a darn writer doing mechanical work:
replay one landed darn task whose diff is a repeated edit across packages, control against Code Mode,
and compare turns, cache-read tokens, cost, wall time and scripts that ended in an error.
Before it runs, one Code Mode session checks that lym's shell guard still refuses a guarded command run inside a script.

## Cap

- One input: one landed darn task.
- Two writer runs: one control and one Code Mode, each once, on the darn writer's default model and effort.
- One guard-check session before them.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-17): The bench input is 888969e7

Decision: The input is the landed change 888969e7 (os.Executable to selfexec.LymBinary at the pins of about 14 packages, 68 files), not bolt-dir-rename.

Rationale: Its repeated edit is Go code across packages that the gate checks, which is the darn work Code Mode is meant to help; bolt-dir-rename is mostly docs and was itself made by a one-shot rename program, so it measures little writer work. The new helper code in 888969e7 is a small share and is the same for both arms.

### Added after Discussion (parent, 2026-11-17): The writers run in local clones outside the hub

Decision: The two writer runs and the guard check run as the advisor bench did (docs/research/loom/advisor-model-bench-2026-11-12): local clones of this repository at the input's base commit under .scratch, outside the hub, driven by lym shuttle run on the darn writer's default model and effort, with no pair or worktree created.

Rationale: The bench needs a fresh tree at the base commit for each arm, and the method is already proven in this repository without touching the hub's pairs.

### Added after Discussion (parent, 2026-11-17): Spend cap for the bench

Decision: About $5 to $10 per writer run is accepted; the whole bench stops at $25, and a run that would pass that is not started.

Rationale: Two writer runs and one small guard check fit well inside the cap, and the cap bounds a runaway arm.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
