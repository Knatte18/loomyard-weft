# shuttle-state: session state read from hooks, in shadow mode

All three batches reported `status: done`, covering cards 1–18.

## Batch 01-deny-table (head `64761f249`)

- Lyx's standing denies are declared in one table that drives both the hooks and the notice, and a `python` Bash deny is added, switched by `claude_deny_python`.
- Hook lines are parsed into provider-neutral session signals with hook-side times.
- Batten's events-file activity now comes from the newest Stop or ask stamp instead of the file's mtime.
- The UserPromptSubmit, StopFailure, Notification and SessionEnd hooks are installed, with a stamp before every recording hook's payload.
- A pure state reducer folds signals and facts into a session state, cause, since and history.
- No deviations.

## Batch 07-claude-prober (head `a79951588`)

- Claude's process liveness is read from the session registry, and its interrupt marker from the transcript.
- A files-only state reader covers every running record, and one run is read through the Runner.
- The read-only `lyx shuttle state` verb is added with its sandbox scenario.
- A held turn end shows as a held wait that batten's quiet notice ignores.
- Payload-reported background shells stay a wait, and a gated run with outputs finishes at once.
- Webster's leftover-shell warning, note and summary are true for both shell kinds.
- No deviations.

## Batch 13-wait-shadow-log (head `d0fc59ea3`)

- The session state and its disagreements are logged beside the wait loop's classification.
- The orch session's state is logged beside the orch watcher's idle probe.
- Each run's state is carried and logged where batten decides a quiet or API-error notice.
- The corpus states each case's session states, and `fork-killed-api-error` gains a StopFailure turn end.
- An `llm`-tier scenario captures real UserPromptSubmit, Notification and SessionEnd payloads; it compiles under `go vet -tags llm` but was not run.
- A regression pin shows a Master's stuck outcome is never reported as a question.
- Deviation: `internal/shuttlecli/modelpin_test.go` was changed because the spawn-site count it guards rose by one with the new `llm` scenario.
