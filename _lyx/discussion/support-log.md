# Support log — timing-pause-as-phase

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- A new wait kind `pause`: waits stay inside their phase's time, so the pause would still count under `plan`, which is the reported defect.
- A sleep-like `paused_seconds` figure taken out of every phase and wait: it needs a new field per run and per bucket and a new markdown column, where a phase gets all three from the existing table.
- Reading the loom status's `state` for the in-flight trailing interval: landed runs can only be read from batten's change entries, so both read the same source.
- Counting batten's own `pause requested` stops: batten pausing stops only its watch, while the child keeps running.

## Review rounds

_No rounds yet._

## Question ledger

- Is the issue's pause the child shed's `paused` state or a hub-wide tender pause? Self-picked: the child state, since batten's notice names a producer and tender pauses leave no git record.
- Where does the new phase sit in report order? Self-picked: after `land`, before `other`, so `other` stays last.
- Does paused time count as active? Self-picked: no, taken out with the waits through one union.
