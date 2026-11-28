# Gate tier timeout counts time spent in hibernation and fails a run that did not hang

`gate-timeout-counts-suspend` · Note · bug, gate, hibernate

Imported from [issue #295](https://github.com/Knatte18/loomyard/issues/295).

## What happened

The machine hibernated from 2026-11-27 22:47:54 to 2026-11-28 08:52:17 while `lym gate` was running tier1.
On wake, the gate killed tier1 as past its 30-minute limit.
The timeout report asked the reader to find a hang or to raise `gate.tiers.tier1.timeout_min`.
Neither applied, and only the system journal showed the cause.

## Why it is abnormal

The tier timeout counts wall-clock time across a hibernation, so time in which the tier could not run is charged against its limit.
A run that was not hung fails, and the report sends the reader after a cause that is not there.

## What to do

- Exclude suspended time from the tier timeout.
- Or detect a suspend during the tier run, say so in the report, and rerun the tier instead of reporting a timeout.

Provenance: task `discussed-payload-map-race`, note `gate-fixer-darn-write.md`.
