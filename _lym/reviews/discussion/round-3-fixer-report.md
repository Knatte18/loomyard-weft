---
dispositions:
  - id: F13
    disposition: fixed-differently
    reason: The proposed test, clear when the row's newest history entry is its reroute entry, misfires when a second send-back is issued inside the Call that followed an earlier reroute and the run crashes before shed persists the second reroute. The newest entry is then still the first send-back's reroute, so the resumed row would clear the second mark and drop that send-back, the same loss F13 names. The fix keys the clear on a reroute entry whose output is the mark's own pr-findings file, which the row returns as its Stuck output and rerouteStuck stores as the entry's Output.
---

## Changes

- **F13, fixed differently** (`_lym/discussion/decision-record.md`, D4).
  - The send-back mark names its `pr-findings-<n>.md` file.
  - On a clean tree the row returns Stuck with `Reroute` set and the mark's file as its output, and clears nothing; shed persists the reroute entry with that output.
  - A new paragraph moves the clear to the start of the row's next `Call`.
    The row reads the run's shed status through a reader the landing wiring injects, since `ShedProducer.Call` takes only a context (`internal/shedengine/producer.go:90`).
    It clears the mark, fix-here count, `parent_review_asked` and session id only when its newest history entry is a reroute whose output is the mark's file.
    A re-step, a crash before the persist, the dirty-tree resume and the cap-halt resume keep all four and reroute again.
  - The fix-here paragraph says the count clears only on the row's entry after a persisted send-back reroute, and lists the cap-halt resume among the cases that keep it.
  - The Bound now says a cap-halted send-back keeps the mark and the count, and its resume reroutes it within the reroutes the resume grants.
    That replaces "has cleared the count too, so each resume opens a fresh allowance".
  - The session-id criterion now says the entry after the persisted reroute clears the id.
    The re-entry criterion now covers the row whose newest entry is its send-back's reroute, the re-step, the crash-resume and the cap-halt resume.
  - Not made, as proposed: a test on the newest entry's outcome alone; see the disposition's reason.
- `_lym/discussion/support-log.md`: a Round 3 entry for F13.

## Self-check

- fix-consistency, LOW: the question ledger still said the reroute clears the session id. Fixed: the ledger entry now says the row's entry after the persisted reroute clears it.
- fix-consistency, LOW: D4's Rationale list of new parts did not name the status reader. Fixed: the list now names the row's shed status reader.
- fix-proposal, LOW: a parent's `lym loom goto` during a marked halt leaves the mark, so the row sends the same file back a second time, and nothing says so. Fixed: D4's Bound now says a goto keeps a pending mark and the row sends that file back once more within the reroute cap, and that `lym loom resume` is the way forward a marked halt names.
- fix-claims: no findings.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
