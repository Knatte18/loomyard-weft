# Round 4 fixer report

All seven findings are fixed in `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`.
No git command was run, and nothing outside the two target files and the two round files was touched.

## Changed

- **F1** (D4, Goal, Open risks, Acceptance criteria): batten suppresses its awaiting notice only while its watched marker stands for the run; it sends when the driver strand has ended, the 3 minutes passed with no report, or the child is still awaiting with no decision 10 minutes after the report's file time.
  The 10-minute wait is a constant. The `Bound:` line and both Open risks now state that bound instead of pointing at the report.
- **F2** (D7, Acceptance criteria, support log): the claim that the chained `set-option` fails on a shared window name is removed.
  The op reads `remain-on-exit` back from the new window by id and kills a window that does not read `off`.
  The tmux 3.6 run behind this: a duplicate exact name exits 0 and sets the option on the older window.
- **F3** (D7, Acceptance criteria, Notes for the plan writer): the strand window resolves from a present Selvage pane (a dead corpse included), then a present strand pane, then the current window; the pane-generation read stays session-scoped and outside the seam.
- **F4** (Constraints, D7): `refusal-spec.md` lists batten refusals out of scope, so no spec row lands; both refusals name a way forward and a test reaches each.
  The failed-creation refusal names `lyx batten run <slug>` in a terminal.
- **F5** (D4): `since` is held by the wait, re-read from the file at a batten start; the state-changed and driver-dead keys are the state and `since`, and the quiet and api-error keys stay D5's activity.
- **F6** (D3, Acceptance criteria): the baseline precedes the read that defines the call's starting state, the spawn arm's second read included; the status decode also runs at every `notice_probe_s` check whatever the modification time says.
- **F7** (D3, D4): the not-parked resume retry logs once per episode and ends when the driver parks or the state changes; the no-strand notice retry ends with its episode or at the `Done` return's last attempt.
- Support log: three rejected alternatives added, one reworded, Q18 updated, and Q25–Q30 added to the ledger.

## Deferred

None.

## Focus departures

None.

## Verification

No test or gate was run here; the mechanical gate over this round's output is the loop owner's.
The H2 sections of both files are unchanged and in their original order.
