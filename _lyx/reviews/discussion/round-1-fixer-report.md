# Fixer report — discussion, round 1

Every finding's premise held against the code: `tenderengine.Status` reports `shuttleengine` state names while `still_busy` and `refused` live only in `PauseReport`; `loginctl --help` on this machine lists no sleep verb and `systemctl --help` lists `sleep`; `commandPositionPattern` covers the prefixes F4 names; `tendercli`'s `defaultGraceSeconds` is 10.

## Changes

- F1 (fixed as proposed): Decision 6's confirmation is one rule over the status state names for every pair and prime session, the orch included: `idle-done`, `idle-stalled`, `idle-fresh`, `asking` or `dead`, so `busy` of any cause and `unknown` withhold it.
  The pause report's `still_busy` and refused lists feed only the first reading.
  The acceptance criterion now lists the same states and adds `unknown` withholding.
- F2 (fixed as proposed): confirmation requires a live hub-scope entry, owned or not, and the pre-hibernate check requires one, ending the request `cancelled` when none stands.
  The pre-existing-pause criterion now covers an operator `lyx resume` before the call.
- F3 (fixed as proposed): Decision 10 drops `loginctl` and denies `systemctl` with `sleep`, `hibernate`, `suspend`, `hybrid-sleep` and `suspend-then-hibernate`; the criterion names `systemctl sleep`.
  The question ledger's deny answer is updated to match.
- F4 (fixed as proposed): Decision 10's bound states the command-position match, flags between `systemctl` and the verb, and what passes.
- F5 (fixed as proposed): Decision 3 refuses an agent caller on `watch`; Decision 5 has `watch` act only for a non-terminal request whose recorded pid and start time are its own, read under the request lock so it sees the record `request` writes in its hold.
  Criteria added for `watch` with no request or beside a live daemon, and for an agent refused on `watch`.
- F6 (fixed as proposed): Decision 4 records pid and start time in the lock hold and defines a live daemon by both; `request`, `cancel` and `status` use that test.
- F7 (fixed as proposed): Decision 6 names tender's default grace of ten seconds; the Auto-mode assumptions line lists it as fixed in code.
- F8 (fixed as proposed): criteria added for `--after` with `--after-all`, `systemctl` missing from `PATH`, a vanishing status file and a request over a terminal request.
- F9 (fixed as proposed): Decision 4 keeps only the hub-scope departure, its reasons merged into the support log's machine-level rejected alternative; the Rationale lines of Decisions 7 and 10 are deleted; the 2026-10-13 incident moved to the support log's interview step 1 and out of the polkit risk.

The support log also records the round under Review rounds and its confirmation ledger answer now matches Decision 6.

## Disputed

None.

## Deferred

None.
