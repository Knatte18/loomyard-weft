# Support log — dobby-start-failures

## Interview

A darn run holds no interview; the board entry is the whole statement of the change.

## Rejected alternatives

- Refuse the board entry when it is created or promoted with a slug that has a finished run record: the board engine reads no run records, and the record can appear on a machine after the entry was released, when the prime syncs. Dobby is where both meet.
- Keep the done record as a refused start with the doubling backoff: after `lym batten cleanup --apply` the task would wait out up to an hour of backoff with an unchanged entry. A hold re-read every tick starts it on the next tick.
- Fix the misread in `firstPassExited` only, by checking whether the status file existed before the pass: the task would still be spawned and fail every backoff interval; the hold avoids the spawn.
- Keep failing-pass counts for done runs across ticks: with D1 no pass of a done run is spawned from the start step, so the count would have nothing to count.

## Review rounds

_No rounds yet._

## Question ledger

- Where is the finished record handled, at release or at start? Self-picked: at start, in dobby (D1), since only there are both the board entry and the run record read.
- Does the hold back off? Self-picked: no; the record is re-read every tick (D1).
- Why did the notice come late? Resolved by reading the code and Neve's dobby log (D4).
- Where did the other machine's record come from? Resolved from the board history: a same-slug task ran on Neve on 2026-10-28 (D4).
