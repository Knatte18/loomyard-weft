# Support log — landing-trailer-text

## Interview

A darn run holds no interview: the writer worked from the board entry's operator decision of 2026-10-17 and a reading of the code.

## Rejected alternatives

- Reading engines from the per-run shuttle directories alone: the orphan sweep removes a directory once its strand is gone, so most sessions' records are gone by Finalize.
- Recording engines in the shed status or seed: those are tracked records owned by other packages, and every session spawner would need a new write path, where shuttle's Start already sees every engine.
- Reading the model spec of each configured role at landing: the config names what may run, not what did, so a run that used only one engine would still name the others.
- Putting engine lines before the fixed lines: the board entry lists the fixed lines first.

## Review rounds

_No rounds yet._

## Question ledger

- Where does the "engines that ran" fact come from? Self-picked: a shuttle-owned ledger at the run directory root, unioned with the surviving run records.
- What happens on an unknown `engine_trailers` key? Self-picked: `LoadConfig` refuses it, naming the key.
- What does a ledger read error do at Finalize? Self-picked: Finalize returns an error before any parent-side mutation, like a description parse failure.
