# tokencount: read each message's usage from its last transcript line

`tokencount` under-counted subagent (fork) output tokens.
A streamed API message appears on several transcript lines, and `tokencount` kept the first.
In a subagent transcript the earlier lines carry a partial `output_tokens` (6 on a thinking block, against 4,546 on the message's final line), so fork costs came out low.
One `twill-fork-timing` bench run's forks cost $1.49 by `tokencount` against $2.09 counted from each message's last line, another $1.15 against $1.62.
Main-session figures were unaffected.

## What changes

- `countFile` in `tools/tokencount/count.go` collects a transcript's usage lines first, then counts one line per message key: the key's last line.
  Messages keep the order their keys first appear in, so a fork's start context is still its first message's.
  The context now comes from that message's last line, and the peak context takes each message's last line.
- The lines dropped within a transcript add to the run's `Duplicates` count, so its meaning is unchanged.
- `readTranscript` in `tools/tokencount/orch.go` applies the same rule to orch transcripts.
  A message's timestamp, which decides whether it falls inside the tally's span and which run it is charged to, is now its last line's.
  The orch tally has no duplicate counter, so its dropped lines are not reported.
- The shared helpers `isUsageLine`, `messageKey` and `lastLinePerMessage` replace the id logic that was duplicated in both paths.
- `tools/tokencount/main.go` documents the rule next to the existing once-per-message-id note.

Figures from earlier `tokencount` runs are low for forks and are not corrected by this change.
That covers the twill benches' fork costs, `docs/research/token-usage-by-role.md` and the `model-downgrade` estimates resting on it; they need a re-run to pick up the new counts.

## Tests

`TestCountRun` now writes a message over two lines with differing `output_tokens` and asserts the last line's figure and the `Duplicates` count.
`TestCountOrch` writes a prime message over two lines, the first with a partial count and a timestamp outside the span, and asserts the message is counted once with the last line's tokens.
