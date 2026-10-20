# Support log — watchdog-reap-test-xdr

## Interview

A darn run holds no interview; the writer read the board entry and the test file and made every call itself.

## Rejected alternatives

- Scale `compressedReapTiming`'s `DiscoveryCycle` up for every step: it slows every step on every host, while a polled deadline costs a fast host nothing.
- Widen only `NeverEnteredOrphanIsStillReaped`'s wait: every other reap wait in the file has the same cycle-derived window and the same exposure.
- Lower `OrphanGoneCycles` in the test timing: the step would stop proving the consecutive-cycle rule.

## Review rounds

_No rounds yet._

## Question ledger

- Fixed floor or scaled timing, as the board entry offers both? Self-picked: a fixed named bound, since the waits poll and a scaled cadence would slow every host.
- Does the `LoopStaysLiveDuringReap` fresh-session wait change too? Self-picked: yes, it waits for an expected success through the same spawning path.
- Does the early-reap sleep change? Self-picked: no, a stall makes it pass, never fail.
