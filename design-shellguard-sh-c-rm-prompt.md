# A shell -c script that removes files stalls a run on a permission dialog

`shellguard-sh-c-rm-prompt` · Task · bug, process · running · Code-Burler-Review

## Seen

2026-11-19: in `hardener-campaign`, the verify fixer fork of Webster ran a loop inside a shell -c script that made a temp dir under the system temp root and removed a subdirectory of it by a variable path, to reproduce a mkdir race.
Claude Code showed a dialog saying the shell -c script removes files and could not be checked, and the run stood still until the orch answered the dialog by hand.
The session rules already refuse a removal by a computed path and writes to the system temp root, but the guard does not look inside the shell -c script, so the command reached the dialog instead of a refusal with a way forward.

## Idea

`internal/shellguard` parses the script of a shell -c call like a top-level command and applies the same refusals, so the agent gets a refusal naming the way forward and no dialog stalls the run.
