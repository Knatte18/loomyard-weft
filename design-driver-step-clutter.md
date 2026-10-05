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
