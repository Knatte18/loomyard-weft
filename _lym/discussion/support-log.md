# Support log — transient-halt-recovery

## Interview

A darn run holds no interview; the writer read the board entry, the peer runs and the code in `internal/gitexec`, `internal/reedengine`, `internal/shedverbs`, `internal/sheddriver`, `internal/shedtransient`, `internal/shedadapters` and `internal/loomcli` and decided alone.

## Rejected alternatives

- Retry EAGAIN in fabricengine's own call sites: rejected because fabricengine runs git only through gitexec, so the exec core covers every site at once.
- Class every died agent as transient: rejected because a died agent with no host shortage behind it is a real failure, and shedengine's admission test asks that a re-run plausibly succeed.
- Surface the tmux error in the halt note only, without a retry: rejected because the step's own re-step is the bounded way past a momentary shortage.
- A new `stderr_file:` field in the halt note: rejected because the trace-file field falling back to the captured stderr gives the reader one file to open without a format change.
- Write failed over a run the dead child left paused: rejected because the pause was recorded deliberately and the next step resumes it; the stop's error still reaches the envelope and the halt note.
- A new leaf package for the refusal record: rejected because proc is already stdlib-only and imported by both spawners.

## Review rounds

_No rounds yet._

## Question ledger

- Should a died chair after a tmux EAGAIN be transient? Resolved by self-pick: yes, when the step's process noted an EAGAIN refusal within two minutes, so the loop re-steps it once.
- Does lym itself spawn enough at the end of Darn-Write to exhaust the process limit? Resolved by self-pick: no change here; the slice was filled host-wide by leaked MSBuild nodes, which board entry `gate-robustness` owns.
- Which stderr lines does the interrupted cause carry? Resolved by self-pick: a Go crash's first lines when present, else the last lines.
