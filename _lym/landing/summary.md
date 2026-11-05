# Seat start reads the skill-load turn's end before it sends the prompt

## What failed

A burler-review seat died twice in a row at start with `shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: ... the pane is not idle`.
The pane showed the skill-load turn finished and an empty prompt.

## Cause

`loadSkillsThenPrompt` moves the run's prompt offset past the load turns' ends before it sends the prompt, so `Wait` never reads a load turn end as the run's own.
The readiness reading the send's idle wait takes folds only the signals past that offset.
Right after the load turns there are none, so the fold settled `unknown` with cause `no-signal`, and the reading fell back to the pane alone.
That fallback is stricter than the hook reading: on Claude it also requires `IdleSession`, which reads the "esc to interrupt" hint anywhere in the capture as a running turn and needs the box rules found around the last caret.
It refused for the whole send-ready window, so the wait gave up and the seat start tore the run down.
The retry seat started into the same state and failed the same way, so the refusal was deterministic, not a race the existing re-poll could outwait.

## Change

`readinessState` in `internal/shuttleengine/readiness.go` folds the whole events file when no signal lies past a non-zero prompt offset.
The session did nothing since the offset was set, so its state is the one the last signal before it gives: here, the load turn's end.
The prompt send now judges that state and the input box as every other ready reading does, and the pane fallback stays for a truly unknown state.
`sessionReadiness` serves both the verified send's idle wait and `Runner.SessionIdle`, so both read the same state.
`Wait` and the files-only readers still fold past the offset, so no load turn end reads as the run's own.

The package doc of `internal/shuttleengine` states the new fold.

## Design decisions

- The fix is in the readiness fold, not a longer re-poll: the send already waits `send_ready_timeout_s` before refusing, and the second attempt failing alike shows waiting longer would not clear it.
- The board entry's suggested retry inside the seat start is left out for the same reason: a retry would start into the same reading and fail again.
- The original startup captures were gone with the earlier task's worktree, so the exact pane line the fallback refused on is not pinned; the fix removes the fallback from this path whichever line it was.

## Tests

`hookStateSkillEngine`, the hook-reading test engine, no longer reads a capture holding the load message as idle, so its pane fallback refuses throughout.
`TestStartup_SkillLoadTurns`' hook-state row and `TestRunner_Start_TypesTheSegmentColorAfterStartup`'s hook-state row now deliver the prompt only through the hook fold.
Without the fix, the prompt send in both rows reads `no-signal`, falls back to the refusing pane and fails with `ErrSessionBusy`.
