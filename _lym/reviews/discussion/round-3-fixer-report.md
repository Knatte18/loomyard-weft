---
dispositions:
  - id: F37
    disposition: fixed
---

# Round 3 fixer report

## Changes

- **F37, fixed.** The premise holds at HEAD.
  `appendHistory` in `internal/shedengine/run.go` appends nothing for an empty outcome, and the error case persists `failed` with the history unchanged.
  `StopKey` (`internal/shedrun/settle.go:13`) is the producer, the history length and the state, so two failed stops at one row share a key.
  - D3 now explains the gap and adds a stop identity, kept in a stop record the driver writes through `internal/state`.
    The driver opens an identity when a step ends in a stop, a restarted driver takes over a matching open stop, and every settle record carries the identity, the hand-back and arbiter `restep` records among them.
  - D3's count is now the distinct stops at the row, each counted once through its halt entry, its handed-on Awaiting entry, or, for a stop with no entry, its identity.
    A settle record adds no stop of its own.
    The nothing-count bullet now says that such a record excludes only a stop the list already excludes.
  - D7 records the four facts on the halt's entry when the halt has one, and on the driver's stop record otherwise, never on an earlier verdict's entry.
    `ChildReading` reads them from wherever the halt recorded them.
    The `halt:` key is now the row and the stop identity, so every stop gets its own key, a fold and a second failed halt at the row among them.
    The history index and `repeats` were dropped from the key: the identity alone already makes each stop's key distinct.
  - D10's first stall reads `settle: orch` from the entry or the stop record.
  - Criteria:
    - D3's first criterion names a stop with no history entry.
    - D3's second criterion adds two cases: the three-failure launch reaching `settle: operator`, and a restarted driver keeping the stop's identity.
    - D7's criterion adds two cases: a failed-state hand-back recording the four facts on the stop record, and a second one getting a new key and a second severe post.

## Self-check

- fix-consistency, LOW: D8's dedup criterion did not name a second failed halt with no entry. Fixed as proposed: the criterion now names it beside a fold.
- fix-consistency, LOW: D3 did not say the stop record holds the open stop's halt entry index and D7's four facts. Fixed as proposed: D3 now says so.
- fix-proposal and fix-way-forward, MEDIUM, merged: the record's open stop never closed, so a driver that died between the step's `failed` write and its record write took over the previous stop's identity and merged two stops.
  Fixed as proposed: the driver closes the open stop before it calls a step, and a restarted driver that finds the record closed opens a new identity.
  D3's criterion now covers a driver that died in that window.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
