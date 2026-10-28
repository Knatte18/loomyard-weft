# Report the mid-turn creel trust spike

Adds `docs/research/creel-midturn-trust-2026-10-28/report.md`, the result of running a creel-style mid-turn post five ways; no lym code changes.

A `PostToolUse` hook returned `decision: "block"` with the scope change as `reason` plus a `systemMessage`, once, during a five-step task, on Haiku 5.5 and Sonnet 5.5 with and without one appended system-prompt sentence, and on Codex `gpt-6-luna` at `low` effort with `additionalContext` plus `systemMessage` and the sentence.
Each case ran once as a reed strand, with turn ends read from a `Stop` hook.

## Findings

- Haiku refused the post without the sentence and followed it with the sentence, so the sentence decides for Haiku; with it, Haiku placed `reviewed` before the last step instead of at the end.
- Sonnet took the post at once with and without the sentence.
- Codex took the post at once with the sentence; the case without it was not run.
- The operator saw the post in every case: three lines under the tool result on Claude and one `↳ Hook` line on Codex.

## Design decisions

- The Claude throwaway directories hold no `CLAUDE.md`, so trust comes only from the channel and the sentence, which the first channel spike did not isolate.
- Codex used the channel the first spike found it takes, not `block`.
- The Codex run accepted its folder trust prompt, which wrote one trust entry to `~/.codex/config.toml`.
