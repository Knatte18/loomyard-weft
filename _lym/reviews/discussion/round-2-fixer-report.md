# Fixer report — Discussion round 2

## Changes

- F1 (MEDIUM), fixed differently in part.
  As proposed: D4 gives each report item an id `P1`, `P2` and so on;
  D7 tags each kept or merged parent finding `origin: parent` and lists the ids it covers in a new `parent_ids` finding key;
  a merged finding keeps `origin: parent`; a review missing an id fails the review gate, naming the missing ids.
  Differently: the check is a gate entry listed after `ReviewGateEntry`, which runs when the chair reaches done,
  rather than a check after the table ends, so a failure re-prompts the chair.
  The entry never passes on its cap (`internal/shuttleengine/gate.go` `PassOnCap`), so a review that still misses an id is never accepted.
  It also closes a still-open ask under the lock before it checks.
  A check that runs before a separate close lets an answer land `done` between the two, which is the loss F1 names.
  The old D7 criterion asked a test to check LLM output.
  It is now a gate criterion, plus a Review criterion on the stencil (severity default, merged origin);
  a not-sent round with the key off gets its own criterion.
- F2 (MEDIUM), fixed differently.
  The close moved into the coverage gate entry (D5), so it always runs before acceptance;
  an accepted review therefore never leaves its round's ask open, and the "driver dies between review and close" window is gone.
  The copy (D4) runs in one locked step at acceptance, on a fresh table and on the hand-on path (`burlerreview.go:257-259`),
  copying a report not yet copied.
  The proposed close at acceptance was not made: after the gate's close it has no state to act on.
  Criteria updated: copy at acceptance, the hand-on copy and refused later answer, and close before acceptance.
- F3 (MEDIUM), fixed as proposed: D5's await `ready: true` answer carries state, `not-sent` reason and report path;
  D7's part is filled whenever the ask record holds an ask for the round's seed and round, whatever the key reads.
  New D5 criterion for the answer's content.
- F4 (LOW), fixed, with the self-check's change to the proposal (see Self-check).
  D4 splits the refusals: a non-latest `--ask` while the latest is `pending-post` or `open` names the latest ask;
  every other refusal names `decision add --by parent` and who runs it.
  D6's bound says a fork of an orch admitted only as the stored strand cannot run `decision add`.
  Criteria updated.
- F5 (LOW), fixed as proposed in D2.
- F6 (LOW), fixed as proposed in D2's seed paragraph.
- F7 (LOW), fixed as proposed under D5's await bullets.
- F8 (NIT), fixed as proposed in D2's facts list.
- Support log: Round 2 entries and two ledger picks added.

## Self-check

- fix-consistency, MEDIUM, await never answers ready for an ask the gate closed early: fixed as proposed (`timeout` in the ready list, criterion).
- fix-consistency, LOW, timeout-refusal criterion not naming the parent: fixed as proposed.
- fix-consistency, LOW, new picks missing from the ledger: fixed as proposed (two ledger entries).
- fix-consistency, LOW, `parent_ids` format key has no criterion: fixed as proposed (Review criterion).
- fix-consistency LOW, fix-claims MEDIUM, fix-way-forward MEDIUM, copy archive clause unreachable after acceptance: fixed as proposed; the clause and the criterion's archive half were dropped.
- fix-proposal, MEDIUM, "answer the latest ask" sends stale findings: fixed as proposed; the refusal says to review the record again under the latest ask's request.
- fix-claims LOW, fix-way-forward LOW, no orch to name when the parent does not resolve: fixed as proposed; the refusal names the operator's unnamed shell then, as `decision.go:685-698` does.
- fix-claims, LOW, coverage entry order unstated: fixed as proposed; listed after `ReviewGateEntry`.
- fix-way-forward, LOW, coverage gate failure's way forward unstated: fixed as proposed; the row fails with the gate's reason and a restart runs a fresh table.

## Disputed

None.

## Deferred

None.

## Rulings

None.
