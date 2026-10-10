# Driver context stays under 100k: Go loops the steps, quieter step output

`driver-context` · Note · enhancement, shed, stencil, loom

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
