# Report: creel mail delivery through a tool-boundary hook

Adds `docs/research/creel-hook-delivery-2026-10-25/report.md`, the measured answer to whether a `PostToolUse` hook can hand a strand its creel mail in full.
It informs note `creel-carry-agent-mail` and is run before creel becomes the one router.
No production code, settings or plugin changes; the diff is the report and two raw logs.

## What the report finds

- Claude, mid-turn: every message posted during one long turn arrived at the next tool boundary, and the 40 s tool call ran to its end.
  The wait is the rest of the running tool call, which is also what SendMessage does.
- Claude, idle: the pane path claimed a message 19 ms after post, and the receiver acted 3.0 s later.
- Exactly-once: with the hook and the pane path racing for 146 messages, each was claimed once (130 by hook, 16 by pane).
- Cost: the no-mail hook adds about 0.6 ms over a bare process start and no tokens; claiming one message takes about 3 ms.
- Receiver: it read every `[creel]` line as a strand message and refused one operator impersonation.
  It answered by `lyx creel post --reply` only when its `CLAUDE.md` said to.
- Recommendation: build the Claude path with a per-strand flag, a flag-first hook and an atomic claim; keep idle-pane delivery; put the reply instruction in the strand prompt; treat the loss of the reminder guard as open.

## Design decisions to review

- Creel's side was simulated with shell scripts under `.scratch/hooktest/` (untracked), not the daemon.
  Exactly-once is therefore shown for the file protocol (claim by atomic rename) only, and the report says so.
- The Codex part is reported as not tested.
  The account hit its usage limit during the run, so the report states that, lists what the 0.164.0 binary exposes, and gives the steps that settle it later.
  This was a deliberate amendment of the Codex acceptance criterion, not an omission.
- Figures from runs whose logs are not committed are marked as such in the report.
  Only the 30-message run and the SendMessage run can be re-derived from the two TSV files.
- SendMessage has a single sample, so the report does not rank it against the hook.

## To check by hand

- Codex: run the steps in the report's Codex section once the usage limit lifts, to learn whether `PostToolUse`, `PreToolUse` or `PreModelRequest` carries mail the model sees.
- Delete the entry the Claude folder-trust prompt wrote to `~/.claude.json` and the one Codex wrote to `~/.codex/config.toml` for the throwaway directory.
