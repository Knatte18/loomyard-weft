# Fixer report — plan, round 2

## Changes

- F1 (LOW), `_lym/plan/02-hostname-name.md` Intent: fixed differently.
  The comparison "unlike `Name`" is gone; the planned doc comment now stands alone and names no other symbol.
  A Bound paragraph limits `FromThisHost` to `lym machine name` and says no other code takes an owner, address or channel name from it, since `Name` and `Names` are the one source of this machine's name and fail on a machine file that yields no valid name.
  I did not keep the proposal's wording "and never as this machine's name" or "serves the machine name verb's own name set alone".
  Card 3's show fills `hostName` with `machinename.FromThisHost` and prints the hostname rule's name as this machine's name when no file exists (rows `NoFileReadsTheHostnameRule`, `NoDirectoryReadsTheHostnameRule`), which matches `resolveNames` in `internal/machinename/machinename.go:131-142`.
  The proposed wording would forbid that use, so the bound now says the verb uses the name as this machine's name only when no machine file exists.
  The self-check's fix-proposal lens raised this change.
- F2 (LOW), `_lym/plan/04-machine-name-set.md` `TestName_Set` rows: fixed differently in name only.
  The row now has the proposed input: file `{"name":"osl46","names":["osl46","neve"]}`, host `osl46`, writes `names` `["neve","osl46"]`.
  I renamed it from `NewNameMovesToTheEnd` to `SetNameOutOfOrderMovesToTheEnd`, because with this input the name is already set and is not new, so the old name would misdescribe the row.
- F3 (LOW), `_lym/plan/04-machine-name-set.md` `TestName_SweepsHubs`: fixed as proposed.
  The `FailedHubIsSweptByTheReRun` subtest is cut.
  The tier 0 row `FailedHubIsReportedAndOthersSweep` now runs a second time with the same `fakeHubs`, the failing hub's `setOwners` now succeeding, and asserts that the hub's request carries the same `Machine` and `Replace` and that the machine file's bytes are unchanged.
  `internal/gitkit#Git` is dropped from card 4's Uses, since the cut subtest was its only user.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-affected-set: no findings.
- fix-proposal, MEDIUM: F1's proposed wording contradicts card 3's use of `FromThisHost` as this machine's name when no file exists. Fixed with my own change: the F1 rewording described above.
- fix-proposal, F2 and F3: no case breaks them; no finding.

## Disputed

None.

## Deferred

None.

## Rulings

None.
