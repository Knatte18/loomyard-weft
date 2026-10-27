# Bench Codex block v2 with a 20-call fork cap on P1, high against medium

Adds `docs/research/codex-v2-p1-bench-2026-10-27/` with `report.md` and `data/`, and changes no product code.
The bench reruns the forked Codex Astra lens fan on plan `P1` three times (one `gpt-6-astra` handler run at high effort, two at medium), with reading block v2 in place of block v1 and the fork boilerplate's cap raised from 6 to 20 tool calls.

## Result

- The forked fan found 6 of the Claude fan's 12 findings at high effort, and 5 and 6 on the two medium runs; both `BLOCKING` findings in two of the three runs.
  Block v1 found 4 of 12 at high and 3 at medium.
- The fan's `F2` (hand-enumerated readers that miss members, such as `templateDefaults`) is found by all three runs; no earlier Astra run found it.
- The union of the two medium runs finds the same 6 fan findings as the high run, at 543k fresh input tokens against 352k; the better medium run alone costs 305k.
- The two medium runs differ by one fan finding (`F1`, the `PATTERN.md` growth order) and three findings overall, which is the spread the other comparisons read against.
- No finding was `false`; the cap of 20 did not bind (at most 12 commands per fork).

## Design decisions

- The harness is the `P1` high bench's, with the root changed, block v2 and the cap number substituted in `build-forkfan.sh` (which exits when an anchor is missing), a run label and effort in `run-forkfan.sh`, one scratch `CODEX_HOME` per label so parallel runs do not collect each other's rollouts, and a new `evidence.sh`.
- The three runs ran in parallel, so the union's wall time is the longer medium run; window points cannot be attributed to a run because the sibling benches drew on the same account at the same time, and the report says so.
- A finding that names part of a multi-member fan finding counts as `shared`, as the earlier benches counted.
- The grader is a Claude session; unmatched findings were checked against the snapshot only where the report says so.

## Docs

A research report with its data; no module, CLI behavior or PATTERN entry changes, so no `doc.go`, `docs/overview.md` or `PATTERN.md` edit applies.
