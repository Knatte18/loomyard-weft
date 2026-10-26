# Shuttle: seat start survives long prompts and tmux EAGAIN, and no held notice for a turn waiting by design

`shuttle-start-hardening` · Task · bug, shuttle, reed, webster

Three shuttle defects in `internal/shuttleengine`, bundled into one darn run: two in the seat start path and one in the held-turn notice.

## Seat start dies on a long prompt (issue #870)

`lyx loom` halted `failed` at the `Darn-Write` producer (anomaly `producer-hard-failure`) after only the Preflight history entry.
The error chain was `shedadapters: Darn-Write (seats): seat run: seatengine: seat "": start: shuttle: start: shuttle: the provider never became ready`, with outcome `died`.
The saved pane capture (`startup-capture.txt` in the shuttle run dir) ends with the tail of the long agent system prompt, wrapped mid-word at the pane width, followed by a closing single quote and a bare shell prompt.
So the provider process was gone and the shell was back in control before the readiness check passed; the strand was then removed.
The shell appears to have echoed or choked on the long, quote-containing prompt text, which hints at the launch command line (the system prompt passed inline) breaking shell quoting or length handling in the tmux pane.
The failure is not retried, so one launch hiccup halts the whole run.

- Check how the shuttle start composes the provider command for a long system prompt (quoting of embedded single quotes, command length); prefer passing the prompt through a file.
- Consider a bounded retry of the start when the outcome is `died`, and keep the head of the pane capture in the halt note, not only its tail.

## Transient tmux EAGAIN read as a dead provider (issue #869)

A `lyx loom` run halted `failed` (anomaly `producer-hard-failure`) at the `Darn-Write` seat producer.
The shuttle start reported "the provider never became ready" with outcome "died", then could not remove the strand because the liveness check failed with `fork/exec /usr/bin/tmux: resource temporarily unavailable`.
The error chain ended in "deliver the prompt after loading skills: check strand liveness: check session", and the halt message told the operator to remove the strand by hand.
EAGAIN on fork/exec is a transient host condition, yet it was treated as the provider dying, so the run hard-failed on a recoverable condition and the cleanup left an orphan strand.

- Retry tmux invocations on EAGAIN with a short backoff in the shuttle liveness and removal paths, and never classify an unverifiable liveness check as "died".
- Retry or defer strand removal so a failed start leaves no strand needing manual cleanup.

This may also be the cause of the `died` outcome above; the run checks whether the two share it.

## Held notice for a turn that waits by design (issue #876)

Merriam started `lyx webster recover-batch 06` in the background and ended its turn to wait for it, as its stencil allows (umbrella-anchor, 2026-10-25).
Each such turn produced a shuttle notice to the parent, "ended a turn without its output files and is held", although Merriam was waiting by design and needed nothing from the parent.
The same notice reached the parent when the operator, in a darn writer's own pane, sent its running shell to the background and asked it questions, and the writer answered and ended each turn while the shell ran (codex-astra-solo-bench, 2026-10-25).

- A turn that ends while the session's own background job runs (the `RequirementBackgroundWake` case) is not reported to the parent as held, nor is a turn that answers a prompt the operator typed in the session's pane.
- Only a turn that ends with no background job and no output files is reported as held.
