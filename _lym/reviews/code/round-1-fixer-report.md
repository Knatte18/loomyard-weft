---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

# Fixer report, round 1

## Changes

- **F1:** took the `CommandRows_IgnoreACommandSetInTheTasksTrackedLayer` row out of the `CommandRows` group and added the serial top-level test `TestBattenIntegration_CommandRowsIgnoreACommandSetInTheTasksTrackedLayer`, which runs the unchanged step on a hub of its own.
- **F2:** narrowed the ordering sentence in `TestBattenIntegration_Rows`'s doc comment as proposed.
  The proposed single line reached the line-length limit and the gate refused it,
  so the comment now says the same in two sentences, with the dirtied prime named in the second.
- **F3:** in `_lym/landing/summary.md` under Measurements:
  - Named the command the six runs used, `lym gate test --gate darn-write ./internal/battencli -- -count=1` (`--no-carry` on every run after the first), and the tier1 phase the figures come from.
    The command and figures were matched against the gate's run records.
  - Replaced the machine-blaming sentence with the earlier measurement's load and the `sar -q` samples around these runs.
  - Replaced "back to back" with the actual spread: three runs before the change, then three after it.
- **F3, beside the finding:** updated What changed in the same file for the F1 move, and noted that the figures predate it.
  The timings were not re-measured.

Commits: one for F1 and F2 together (they share a file), one follow-up for the self-check's comment finding,
and one for the line-length repair of F2's comment.
`_lym/landing/summary.md` is not committed by this round.

Verification: `lym gate test --tier 1 ./internal/battencli` passed after each code change, and the new test shows as PASS in the first run's log.
The default `lym gate test ./internal/battencli` (tier0) passed after the first change.

## Gate tier1 timeout

No code change: the timeout was not a hang.
The tier1 run started at 22:46:31 on 2026-11-27, the machine hibernated at 22:47:54 and resumed at 08:52:17 the next day (`journalctl`),
and the 30-minute limit expired on resume.
The run record shows 100 s of `wall_seconds`, and its log shows tests still passing when it was killed.

Re-run: `lym gate test --tier 1`, without a gate key, finished in 80 s with 545 passed, 598 skipped and 1 failed
(`_bolt/.lym/gate/runs/20261128T075706Z-tier1-2086456.log`).
The failure is `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`, which reports `internal/shedcli.names` as reached by no main package.
This branch does not touch `internal/shedcli`, and the gate that timed out skips that test, so it is left alone here.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW (`posixOnly`'s doc comment described only the step loop): fixed as proposed, in other words; own commit.
- fix-proposal, LOW (the load statement did not say that no `sar` sample falls within the runs): fixed as proposed in `_lym/landing/summary.md`.
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
