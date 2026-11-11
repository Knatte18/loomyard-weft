# Research: three independent proposals for coverage map upkeep, compared

Adds the study `docs/research/speed/testmap-upkeep-2026-11-11/` and its line in `docs/research/README.md`.
The change holds documents only: no code, test, config or PATTERN entry changes.

## What the study holds

- `proposal-fable-a.md`, `proposal-fable-b.md` and `proposal-opus.md`:
  one proposal each from two Fable 5.2 agents and one Opus 5.5 agent, all at high effort, for a smarter way to keep the per-test coverage maps fresh.
  Each agent started fresh with the same brief and saw no other agent's answer.
  The files are the agents' text, unedited.
- `report.md`: the comparison, as the input for a later design task.
  It lists where the three agree, sets them side by side in one table, describes each point where they differ and what it costs,
  gives each proposal's verdict on the note's six starting ideas,
  and lists what each asks of the waiting gate-scope tasks (`testmap-cross-package`, `coverage-test-select`, `gate-scope-coverage`).
  It ends with the measurements the proposals ask for and the questions the design task has to decide.

The report decides no design, as the task requires.
No agent ran a test or a build, so every duration in the proposals is an estimate or a figure read from an earlier record, and the report says so.

## What the comparison found

The three agree on the main line without having seen each other:
the test becomes the unit of staleness, at function grain rather than the note's block grain;
the selector that the gate scope will use also decides what upkeep measures again, called against the prime's HEAD;
the age rebuild gives way to an environment stamp and a sampled audit;
the dobby starts a refresh when work remains, not on a landing count;
and upkeep never makes a gate wait.
All three also name the same limit: a change outside function bodies still empties the map of every package that links the changed file,
and only a narrower miss rule in the selector would avoid that work.

They differ on whether a map carries one commit or one per test entry, on how upkeep gives its gate slot back,
on idle-only against running whenever a slot stays free, and on measuring in the prime worktree against a checkout pinned to a main commit.

## Design decisions

- **One study folder under `speed`.**
  The task asks for one report under `docs/research/`.
  The folder follows the README's form of a study: `report.md` beside the data behind it, here the three proposals in full, so the design task can read each argument whole.
  `speed` is the topic that already holds the test-redundancy and gate-time studies.
- **Fixed section headings in the brief.**
  The brief gave all three agents the same headings so the proposals compare section by section; what stands under each heading was the agent's own.
- **Wider reading than the task names.**
  Beside the files the task lists, the brief let each agent read the rest of the repository and the board entries around the maps, read-only.
  The maps' one waiting consumer is the gate scope, and its decision records state what a map must hold.
- **Opus at high effort.**
  The task names high effort for the Opus agent, which is followed over the standing default of medium effort for a subagent on a model other than Fable.
- **A check of the proposals' claims.**
  The report checks the claims about today's code that the comparison leans on, and the numbers the proposals disagree on, against commit `4ae8978ec`, and lists what it did not check.
  One result matters for the design: two proposals reason from the template's two gate slots, while this hub's `gate.yaml` holds three.
