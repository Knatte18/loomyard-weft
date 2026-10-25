# Darn-Write seat start dies: provider pane drops to a shell prompt after the prompt text

`seat-start-long-prompt-dies` · Note · bug, shuttle

Imported from [issue #870](https://github.com/Knatte18/loomyard/issues/870).

`lyx loom` halted `failed` at the `Darn-Write` producer (anomaly `producer-hard-failure`) after only the Preflight history entry.

The error chain was `shedadapters: Darn-Write (seats): seat run: seatengine: seat "": start: shuttle: start: shuttle: the provider never became ready`, with outcome `died`.
The saved pane capture (`startup-capture.txt` in the shuttle run dir) ends with the tail of the long agent system prompt, wrapped mid-word at the pane width, followed by a closing single quote and a bare shell prompt.
So the provider process was gone and the shell was back in control before the readiness check passed; the strand was then removed.

Why it is abnormal: the provider should come up or fail with its own error.
Here the shell appears to have echoed or choked on the long, quote-containing prompt text instead, which hints at the launch command line (the system prompt passed inline) breaking shell quoting or length handling in the tmux pane.
The failure is not retried, so one launch hiccup halts the whole run.

What to do:
- Inspect `startup-capture.txt` of the named run dir to see the start of the launch command and the first error line, which the note's tail does not show.
- Check how the shuttle start composes the provider command for a long system prompt (quoting of embedded single quotes, command length); prefer passing the prompt through a file.
- Consider a bounded retry of the start when the outcome is `died`, and keep the head of the pane capture in the halt note, not only its tail.

Provenance: task slug `gate-runs-past-failing-tier`; notes: `loom-halt.md`.
