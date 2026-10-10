# Driver steps in only on trouble: Go loops the steps, context under 100k

`driver-context` · Task · enhancement, shed, stencil, loom

Depends on: [`daemon-wakeups`](design-daemon-wakeups.md)

## Goal

The driver steps in only when something goes wrong, and its context stays under 100k tokens for a whole run, since Haiku 5.5 is cheap only below that.
Build idea 1 below: Go runs the steps in a loop and wakes the driver only at a stop (error, interrupt, blocked, awaiting gate, run end).
Ideas 2 to 6 come along where they still apply once the loop is in Go; idea 7 is optional.

`lyx shed step` says too much on success and too little on failure: a successful step prints its envelope plus stderr slog noise the driver must read, while a failed one prints too little, so the stencil has the driver read every step's trace in case a later failure needs it before retention sweeps it.
Turn that around:
- A successful step answers in one short line, and the driver reads almost nothing before the next step.
- A failed or stopped step answers at length: what went wrong, the trace lines that matter, and the paths to read further, so the driver can act on the answer itself and opens the trace only to dig deeper.
- Retention never sweeps the trace of a step that failed or stopped before the driver has handled it, so the driver never reads a trace pre-emptively, and the stencil drops "After every step, read its `trace_file` right away".

The heavy work at a failure goes to a one-shot fork of the driver: reading traces, reports and logs, and writing the distilled report it sends to the parent.
The fork returns only its conclusion and the action to take, so the driver's own context stays clean across many stops.
With the Go loop the driver's context stays small, so a fork that inherits it is cheap and stays under 100k too.

The orch must still be able to tell the driver, in plain words, to stop the run at a chosen point, for example "stop after Batchifier, before Webster", and the driver must hit that point itself.
`lyx loom pause` is not the route: it is too restrictive.
So a stop condition, such as a producer to stop before or after, is recorded in the run's own state by a command, and the Go loop reads it between steps, stopping there before the next producer starts and waking the driver.
Setting it never restarts the loop or interrupts a running producer: the producer in progress always finishes.
A message to the driver still reaches it promptly while the loop runs, and the driver turns such a request into that command, using the producer names the step envelope and the recipe give it.

The Go loop also watches its own steps: a step that never returns is noticed and stops the loop as a failure that wakes the driver, so the hub never has to read panes to find a hung `lyx shed step` (from `verify-watchdog`, #420).

A failed step never parks silently (from `driver-silent-park`, #488).
In `bolt-and-store` (2026-10-10) a Webster-Bouncer step returned `ok:false` with no `kind` and no trace, because a hub config file (`_board/_lyx/config/landing.yaml`) a later deploy added was missing; the driver wrote a drive report, parked with `commit-records --park` and told the parent nothing, while the run stayed `running`.
The hub heard of it only through batten's quiet notice, and `lyx loom resume` answered "the run is already running".
So a step failure without a `kind` is a stop that reaches the parent, as a halted run or a driver escalation, and the missing-config refusal gets its row in the refusal spec.

## Seen

A study of 19 driver sessions (2026-10-10) found that every finished run's driver passed 100k tokens of context, peaking at 112k to 190k.
They cross around API call 80 to 115, mid-run, at Plan-Review, Webster, Webster-Review or Batchifier.
Haiku 5.5 is cheap only below 100k, and the Haiku drivers grow faster per step (4.0k to 4.7k) than Sonnet (2.2k to 3.6k), since they `cat` whole trace files.
There is no compaction, so the peak is the last turn.

The growth is a fixed cost per step, not one big item: each `lyx shed step` cycle takes four API turns and adds 2.2k to 4.6k tokens, over 35 to 47 steps per run.
Of 1.98M tokens of growth across the sample:

| source | share |
|---|---|
| assistant output, thinking included | 24% |
| Read of the step's background output (envelope plus stderr) | 19% |
| trace reads after every step | 15% |
| task-completion notice per step | 12% |
| harness attachments | 9% |
| prompt.md and skill body | 7% |
| background-launch result | 4% |
| `lyx shed status` | 4% |
| Reads of the files a step's `output` names, mostly `decision-record.md` at 13 to 25KB | 2.5% |

The start is about 20.5k (tool schemas about 6k, the skill listing about 5 to 6k, system prompt and CLAUDE.md about 4k), then about 30k after the `scribe:prose` body and the 18.9KB `prompt.md`.

Noise rides along in step stderr:
- the plan gate's informational findings as one WARN line of up to 25KB (`internal/loomshed/gates.go:199`), 172KB across the sample;
- `stencilstore: board copy has drifted` up to 58 times per run (`internal/stencilstore/reconcile.go:236-240`), 106KB;
- full Go SIGQUIT goroutine dumps of about 20KB when a step is killed, 76KB.

Report writing (drive reports, SendMessage, friction) is about 3k tokens per run, so having the driver fork itself for reports would not help: each fork does a full cache read of the parent's context and starts above 100k whenever the parent does.

## Idea

Ranked by whether it keeps a whole run under 100k:

1. Go loops the steps and wakes the driver only at a stop, for example `lyx shed step --until-stop` in `internal/shedverbs/step.go`, keeping each step's trace for the driver.
   It saves 80 to 130k per run, with peaks of about 35 to 50k, and is the only single change that reliably keeps a Haiku driver under 100k.
2. The driver reads a step's trace only on an error, interrupt or stop, not after every step (`contracts/stencils/shed/shed-template-driver.md`); Go keeps the trace of a failed step.
   It saves 25 to 45k per run.
3. Keep slog WARN lines and stack dumps out of the step's stdout and stderr and send them to the trace: a count instead of the full plan gate findings, the drift warning once or at debug, the SIGQUIT traceback to the trace file.
   It saves about 7k per run, 20k on a run with many gate findings.
4. Drop "reading the artifact `output` names when it is non-empty" from the stencil: 0 to 10k per run.
5. A slimmer start: drop `scribe:prose` from `driverSkills` in `internal/loomcli/driverspec.go`, keep the skill listing and unused tools (Agent, ListAgents) out of the driver session, and move the stencil's rare branches to a file read on demand: 10 to 13k per run.
6. A compact envelope from `lyx shed step` and `lyx shed status`, without `remaining`, empty fields and repeated absolute paths: 6 to 10k per run.
7. Lower effort or thinking for Sonnet drivers: about 10k.

Changes 2 to 6 without 1 bring a 45-step run to about 90 to 95k, borderline.
