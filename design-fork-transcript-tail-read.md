# Merriam turn-end read parses every fork transcript in full

`fork-transcript-tail-read` · Note · enhancement, shuttle

## Seen (2026-10-11, fork-wait-noise PR #555 review)

`claudeengine` `holdsPendingToolCall` parses each fork transcript in full at every turn-end activity read of Merriam.
A Master with many batches or large fork transcripts pays that parse on every read.

## Idea

Read each fork transcript backward from its tail and stop at the first tool result or pending call, so the cost stays bounded by the tail, not the transcript length.