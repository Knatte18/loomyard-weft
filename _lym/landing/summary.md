# Bench Codex block v2 with three gap study texts on P2: two medium runs

Adds `docs/research/codex-p2-gap-texts-bench-2026-10-27/` with its report and data; no code, contract or PATTERN file changes.

## What the change does

Two forked Codex Astra fan runs (`gpt-6-astra`, medium, three lens forks, cap 20) on plan `P2`, with block v2 and three texts from the gap study: Proposal 1 and 2's `guard-reach` sentences and fork-reading bullet and boilerplate wording, and Proposal 4's `way-forward` sentences.
The report grades every finding by hand and compares real findings per weekly-window point, and per fresh input token, against the two medium runs of `codex-v2-p2-bench`.

## Result

- The texts did not measurably change the yield: 12 findings in 5 window points over the two runs (2.4 per point), and 2.4 against 2.3 findings per 100k fresh input tokens relative to the earlier medium runs.
- The two new runs differ from each other by 6 findings (9 against 3), so a difference of one or two findings is inside the run-to-run spread.
- Each run found only `F1` of the Claude fan's 13 findings; the `way-forward` fork returned no finding in either run, and run 2 returned `APPROVED`.
- No false finding in either run; run 1 found one new `BLOCKING` defect that holds by reading (an overload delete and re-sign sharing a batch).

## Design decisions

- The harness is copied from `codex-v2-p2-bench` with `ROOT` changed to this worktree, so the committed scripts reproduce the bench; `build-fan.sh` and `build-forkfan.sh` patch the texts and fail when an anchor is missing, and `usage.sh` is new.
- The `guard-reach` text takes Proposal 1's sentences, then Proposal 2's added sentence, then Proposal 1's `Your list`, since Proposal 1 replaces the old `Your list` and Proposal 2 adds before it.
- Proposal 1's reading bullet sits after the bullet on the functions a card changes and their callers; block v2's own "one hop beyond the cards" bullet is unchanged, as Proposal 2 names the fork boilerplate only.
- The first run's window start is its first request's reading (62), not a probe, because a probe would cost budget; window points are whole percents, so each run's figure carries about one point of error.
- The earlier bench's window points are shared with sibling benches, so the per-point comparison falls back on fresh input tokens; the report says so.

## Testing

No Go code changed, so no test ran.
The decision record and support log pass `lym loom validate-discussion`.
