# Bench the explorer subagents of the Discussion and Plan writers

Adds `docs/research/loom/writer-explorers-bench-2026-11-18/`, a research report on whether the explore and lookup subagents of the Discussion-Write and Plan-Write stencils pay for themselves.
No production code, stencil or test changes.

## What it contains

- `report.md`: the share measurement, the bench's cost, time and grades per arm, and the recommendation.
- `keys.md`: grading keys drawn from the review findings of the reference run `pawl-checklist-hidden-in-beam`, committed before any arm ran.
- `share.sh`, `slugs.txt`, `share-rows.tsv`, `share-totals.tsv`: the explorer share of the writers' spend over the landed tapestry runs since 2026-11-03, split by the `agentType` of each subagent's meta file.
- `arms-rows.tsv` and `no-explorers.patch`: the two bench arms' priced transcripts and the stencil edits the without arm ran on.

## Findings

- Explorers are 5.8% of the Plan-Write spend ($18.62 of $318.58, 82 subagents over 23 runs) and 8.9% of the Discussion-Write spend ($3.22 of $36.08, 17 subagents over 7 runs).
- In the bench, the with arms spent $9.37 on writers, forks and explorers against $8.68 for the without arms; the explorers' own spend was $0.38 against $0.56.
- The without arms' writers launched explorers anyway, so the stencil lines did not decide whether a writer used them.
- The grading keys did not discriminate: neither arm reproduced the reference's design, and five of the six keys did not apply.

The recommendation is to keep the explorers.

## Design decisions

- The two Discussion-only arms the board entry asked for were dropped: no lym verb skips Discussion-Write on a plan pair, so each arm is a full run through Discussion and Plan-Write, and each plan rests on its own arm's record.
  The orch recorded this as a parent decision in session mail, not as an `Added after Discussion` entry in the record.
- The cost cap held: the writer sessions cost about $18, and the review and fix sessions are not priced in the report.
- The earlier "21 runs" count of `tokencount` is not reproduced; the report counts the runs of `slugs.txt`.

## Verification

`share.sh` ran over `slugs.txt` and its tables are the committed TSV files.
