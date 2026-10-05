# The LLM driver wraps every shed step in trace ids, temp files and cut

`driver-step-clutter` · Note · enhancement, shed

Seen 2026-10-05 in `lyx-skills-and-stencils`: the LLM driver wraps every step as

```
exec </dev/null; export LYX_TRACE_ID=$(od -An -N8 -tx1 /dev/urandom | tr -d ' \n') && lyx shed step <run-id> 2>/dev/null > /tmp/tmp.XXXX/step-28.json; echo exit $?; cut -c1-1000 /tmp/tmp.XXXX/step-28.json
```

and each background job's completion notice repeats the whole line into the driver's context and pane.
`ly-drive` already says "Mint no `LYX_TRACE_ID`, create no `mktemp` step directory and add no redirects", since `lyx shed step` mints its own trace id and records each invocation under `.lyx`; the driver does it anyway.
It is the same model habit as `agent-stdin-hang` (wrapping a command it has no reason to wrap); the `exec </dev/null;` prefix is that note's stdin hook.

Proposal: make the bare call the convenient one, so the model has nothing to wrap.

- `lyx shed step` prints a short envelope by default (outcome, producer, state, the next action), with the full envelope behind a flag or in its own record, so no `cut` or temp file is needed to read it.
- The driver stencil shows the one exact command to run, `lyx shed step <run-id>`, as a background job, and nothing else.
- Check whether the step could instead block in the foreground within the Bash tool's limit when the producer is quick, so only long producers produce a completion notice.

## Findings (hub, 2026-10-05)

- Every LLM driver since at least 2026-10-03 wraps its steps this way, in every run; `ly-drive` prescribed the wrapper until `083e992cf` (2026-09-30) and has forbidden it since, so the habit outlived the instruction.
- A likely motive: the skill says to read each step's trace, and the model mints its own id to find it, though the envelope already carries `trace_file`.
- Some drivers go further: `test-suite-measure`'s writes a `run.sh` through a heredoc into `/tmp`, the shape behind `agent-stdin-hang`.
- The step must stay a background job: Claude Code's Bash tool caps a foreground call at 2 minutes by default and 10 at most, and a step can block for an agent run of up to an hour.
