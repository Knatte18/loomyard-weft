# Agents stop wrapping commands: stdin without a hook, a bare shed step

`agent-command-wrapping` · Task · bug, enhancement, shuttle, shed, stencil · running · Discussion-Burler

Agents wrap commands they have no reason to wrap.
Two symptoms of the one habit, merged from the notes `agent-stdin-hang` and `driver-step-clutter`.

## Stdin hang

Agents' Bash shells hung on stdin for up to 80 minutes, holding verify gates (`board-model` #361 on 2026-10-04; `orch-cycle-hardening` and `webster-audit-recovery` on 2026-10-05).

Cause (transcript study, 2026-10-05): the model, mostly Sonnet fork subagents, emits a bare `cat > /tmp/x;` or `python3 -;` with no input attached, often as a no-op (`cat > /tmp/x 2>/dev/null; true`); 54 such calls between 2026-09-24 and 2026-10-04.
The stored commands keep their newlines and no layer rewrote them.
No stencil shows the shape, and lyx cannot change the habit.

`3c5bfdb04` gave every Bash call `/dev/null` as stdin through a PreToolUse(Bash) rewrite hook (`bashStdinHookCommand`, `internal/shuttleengine/claudeengine/settings.go`), so such a command ends at once while heredocs, pipes and redirects still work.

- The operator dislikes hooks: each one delays every tool call.
  Give the agent's shell `/dev/null` as stdin without a per-call hook (e.g. how shuttle launches Claude Code, or `BASH_ENV`, checking it does not cut stdin from a script fed through a pipe), and drop the hook if one works.
  A timeout that ends the hang after the fact is not a fix.
- If no hook-free way works and the hook stays: it extracts the command with a `sed` regex taking the last `"command":"` key in the raw payload, so a later `command` field elsewhere in the payload would be rewritten instead; parse the JSON.

## Driver step clutter

Seen 2026-10-05 in `lyx-skills-and-stencils`: the LLM driver wraps every step as

```
exec </dev/null; export LYX_TRACE_ID=$(od -An -N8 -tx1 /dev/urandom | tr -d ' \n') && lyx shed step <run-id> 2>/dev/null > /tmp/tmp.XXXX/step-28.json; echo exit $?; cut -c1-1000 /tmp/tmp.XXXX/step-28.json
```

and each background job's completion notice repeats the whole line into the driver's context and pane.
`ly-drive` already says "Mint no `LYX_TRACE_ID`, create no `mktemp` step directory and add no redirects", since `lyx shed step` mints its own trace id and records each invocation under `.lyx`; the driver does it anyway.

Findings (hub, 2026-10-05):

- Every LLM driver since at least 2026-10-03 wraps its steps this way, in every run; `ly-drive` prescribed the wrapper until `083e992cf` (2026-09-30) and has forbidden it since, so the habit outlived the instruction.
- A likely motive: the skill says to read each step's trace, and the model mints its own id to find it, though the envelope already carries `trace_file`.
- Some drivers go further: `test-suite-measure`'s writes a `run.sh` through a heredoc into `/tmp`, the shape behind the stdin hang.
- The step stays a background job: Claude Code's Bash tool caps a foreground call at 2 minutes by default and 10 at most, and a step can block for an agent run of up to an hour.

Make the bare call the convenient one, so the model has nothing to wrap:

- `lyx shed step` prints a short envelope by default (outcome, producer, state, the next action), with the full envelope behind a flag or in its own record, so no `cut` or temp file is needed to read it.
- The driver stencil shows the one exact command to run, `lyx shed step <run-id>`, as a background job, and nothing else, and tells the driver where the trace is.

## Scope and coordination

Runs alongside `test-suite-prune-core` (owns `internal/shuttleengine`) and `test-suite-prune-rest` (owns `internal/shedcli` and the rest).
Touch only the tests that cover the changed behavior; the hub tells the prune runs to leave those alone.

Done when: no lyx-spawned Claude session carries the stdin hook, or the hook parses JSON if it must stay; `lyx shed step` prints a short envelope; the driver stencil shows the bare call.
