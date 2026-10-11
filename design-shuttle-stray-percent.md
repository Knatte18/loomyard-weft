# shuttle start dies with 'provider never became ready' when the idle pane shows a stray % after the prompt

`shuttle-stray-percent` · Note · bug, shuttle

Imported from [issue #549](https://github.com/Knatte18/loomyard/issues/549).

## What happened

`lyx shed step` halted the `Darn-Write` producer as `failed` (anomaly `producer-hard-failure`) with:

```
shedadapters: Darn-Write (shuttle): shuttle run outcome died: shuttle: the provider never became ready
```

The trace shows the start sequence failing before the provider took any input:

- `shuttle: could not type the segment color` with `the session is not idle ... the pane is not idle; retry once the session is idle`.
- `shuttle: start failed before the provider took its first input; strand torn down`, cause `shuttle: load skills [scribe:prose scribe:code-quality]: shuttle: the session is not idle`.

The pane's last lines, quoted by lyx itself, show a fully started Claude Code UI waiting at an empty prompt, with a stray `%` after the prompt character:

```
❯ %
────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle)
```

The first idle check failed about 70 seconds after the run started, and the skill-load idle check failed about 60 seconds after that, so the pane stayed "not idle" through both waits although the provider was visibly ready.

## Why it is abnormal

The provider was up and at an empty prompt, yet reed/shuttle classified the pane as not idle and gave up, tearing down the strand and failing the whole producer.
The stray `%` is likely a leftover from the shell's end-of-line marker or a partial draft, and idle detection treats any text after `❯` as an operator draft.
A later reflection agent spawned in the same worktree reached `busy` on the same Claude Code version without trouble, so the failure is in the idle/readiness detection, not the provider.

## What to do

- Make the idle check tolerate a lone trailing `%` (or similar non-input terminal marker) after the prompt character, or clear the input line before checking.
- On a not-idle result during start, retry or wait longer, rather than failing immediately with the generic `provider never became ready`.
- Include the pane's last lines in the final `shedadapters` failure message, as the trace already does, so the cause is visible without opening the trace.

Provenance: task `batten-notice-trim`, from `loom-halt.md`.
