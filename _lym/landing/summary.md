# Report the Code Mode and shell-pipelines bench on the Plan writer

This adds a research report and its raw data under `docs/research/claude/code-mode-bench-2026-11-16/`; it changes no production code.

## What the report measures

It replays the Plan-Write row on two landed tapestry runs, `machine-name-verb` and `run-fsm-tracked-records`, in three arms each:

- A is today's Plan-Write.
- B sets `CLAUDE_CODE_EXPERIMENTAL_CODE_MODE=1` on the writer's launch.
- C lifts the read rule so the writer may read and search by shell pipeline.

Each of the six sessions then got one Plan-Burler review round.
No landed tapestry Plan-Write is twill-free, so both inputs are twill runs replayed with `plan_lenses` empty.

## Result

Code Mode shows no gain the sample can tell from noise.
B's writer cost is 5% above A on the small input and 12% below on the large one, its wall time 14% and 2% above, and its plans repeat one grading key fewer on each input.
B made twice the tool calls of A on the large input, but its turns fell by only three, because each REPL call needs a Poll call and the control already batches in the shell.
C cost more on both inputs and made the worst large plan on the keys.
A lookup subagent runs in Code Mode with its session.
The Discussion writer bench the board entry made conditional on a gain is therefore not warranted.

## How the bench set the arms

Arm B names a wrapper script in the pair's `claude.yaml` `binary` key that execs `claude` under the variable, so no production code changed and no user setting was set.
Arm C uses a wrapper that appends an instruction overriding the shared edit directive, since the shared stencil was in use by other runs.
Both wrappers are in `data/`.

## Design decisions

- The orch made the six pairs and the writer ran them, on the operator's recorded decision; the earlier helper-run plan was replaced.
- The grading keys in `keys.md` were committed before any arm plan was read, and each grader subagent read the plans without knowing the arms.
- The report names its limits: one sample per arm and input, the hand-fed answer all three small arms received about `machinename.HostnameName`, Plan-Bouncer sessions of two B pairs that ran in Code Mode, and runs that overlapped in time.

## Side effect on the machine

Answering Code Mode's first-use dialog once wrote `codeModeCaveatsAcceptedV1` and `codeModeAutoModeCaveatAcceptedV1` to `~/.claude.json`; no setting was turned on.
