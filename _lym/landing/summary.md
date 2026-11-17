# Retro of ten landed runs: report, figures and four proposals

Audits the ten runs that landed between 2026-11-10T06:46:08Z and 2026-11-16T11:54:11Z from their records and transcripts.
The change adds a report and the files behind it under `docs/research/retro/retro-20261116-115411/`, and one line for it in `docs/research/README.md` under a new `retro` section.
It changes no production code, test, stencil or file under Bolt, and files nothing on the board or as an issue.

## What the directory holds

- `report.md`: the runs and window, the figures behind each proposal, four proposals with severity, route, file and moment, and the questions the runs did not answer.
- `figures.md`, `tokencount.md` and `forkaudit.md`: the output of `tools/retro/retro.sh`, `tools/tokencount/tokencount.sh` and `tools/forkaudit/forkaudit.sh`, kept whole.
- `sessions.jsonl`, `rows.jsonl`, `refusals.jsonl`, `usage.jsonl`, `facts.jsonl`, `keys.jsonl` and `missing.jsonl`: the tables behind `figures.md`.

The directory's name is kept as it is, because the next `lym retro` reads its stamp from it.

## Proposals

- Stage the code reviewer's diff in pieces that fit one Read (medium, a check in burler's review staging).
- Add a discussion-review rubric line on facts a decision rests on (medium, a rubric line).
- Make the shell guard's refusal say which part of a compound command ran (low, a check, to confirm against the current text).
- Delete or enforce the `cat`, `head` and `sed -n` clause of the edit directive, which every sampled session broke without a fault (low).

## Design decisions

- The reading stayed within twelve moments, listed in the report by run, session and turn.
  Reports of the verify gate were not read, so the report names the fault kind of a gate failure as unanswered.
- Proposal 3 rests on three observed refusals and does not claim the refusal text lacks the statement; it asks to confirm that first.
