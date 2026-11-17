## Goal

```markdown
## Goal

Level 2 of `codex-support-levels` (R2): can Codex do the darn writer's work on real lym code as well as Claude did?
The bench replays only the writer, never a whole run: no other producer runs, and nothing lands or is pushed.

## Why again (operator, 2026-11-16)

The earlier bench on Neve does not count: its Codex session did not get the starting point Claude's writer got.
This bench runs only once the Codex parity tasks of the project `codexengine` (from the Sketch `codex-same-as-claude`) have landed,
so the Codex writer gets exactly what a Claude writer gets.
It runs on Neve; the OSL-1046 run and its sandbox fallback check are dropped.

## Input

The darn light task `ide-beam-readme-preview`, landed as `824eb2b6`.
Its Claude writer's work is the answer key: the landed diff, and the run's records on its records branch (writer cost, review rounds and findings).

## The one run

1. The orch creates one bench worktree from this task's worktree, at the parent of `824eb2b6`.
2. Run the real Darn-Write producer there on the task's body as it stood then, with only the writer's model swapped to `astra[high]`.
   Never hand-build the prompt.
3. Before the session starts, check with `lym shuttle manifest` that the Codex writer's manifest holds every item the Claude writer's holds.
4. Stop the run after Darn-Write, and run `lym gate test` on the packages the diff touches.

Exactly one Codex session; no Claude run, since the landed run is the comparison.

## Read

- Did it finish the work, and does the gate pass on its tree?
- Its diff against the landed one: scope, correctness, comments and prose in the house tone.
- Cost, wall time and turns against the landed writer's.
- Any skill-load or idle-wake fault on the way (`codex-idle-wake-extra-reminder`).

## Bounds

Change no production code and nothing under Bolt; the orch removes the bench worktree when the writer says so.
The report goes under `docs/research/codex/`; the verdict on level 2 is the operator's.
The report opens with the Codex writer's manifest record and the Claude writer's `lym shuttle manifest` output for the same seat, as the project's bench rule says.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-17): Bench pair and who runs it

Decision: The orch created one bench pair, cwb-codex at /home/knatte/Code/loomyard-LYMHUB/cwb-codex, forked from this task's worktree. The darn writer of this task runs everything in it: moves its tree to 055ef994 (the parent of 824eb2b6), runs lym shuttle manifest for the Darn-Write seat, runs the real Darn-Write producer with only the writer model swapped to astra[high] through run_settings, stops after Darn-Write, runs lym gate test on the touched packages, and writes the report. The orch removes the pair when the writer says so. The run waits until the installed lym carries lym shuttle manifest (from 3bbc92a0); the orch tells the writer when it does.

Rationale: A task session cannot run lym fabric add, so the orch creates bench pairs from the task's worktree and the task agent runs them (operator, 2026-11-16). The manifest check is the bench's precondition, and the installed binary predates the verb.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
