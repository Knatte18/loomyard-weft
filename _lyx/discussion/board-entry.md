# Board entry `decision-add-batch`: lyx loom decision add records several decisions in one call and one notice

## Seen (2026-10-24)

To get `gate-runs-once` past PATTERN headroom the orch shortened 20 PATTERN entries.
`lyx loom decision add` takes one `--pattern-entry` per call, so it took 20 calls, 20 commits and 20 full creel notices into the plan session, each repeating the whole decision text: noisy for the agent and the operator watching it.

## Idea

`lyx loom decision add` accepts several decisions in one call, such as `--from <file>` with one entry per decision or repeated `--pattern-entry`/`--pattern-text` pairs, writes them in one commit and sends one short notice naming the new headings and pointing at the record, never the full texts.
