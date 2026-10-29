## Interview

A darn run holds no interview.

## Rejected alternatives

- Setting `go_parallel`: retired, the load drops it.
- One shared logger edit for both runs: the second run would hit the test cache.
- A comment-only logger edit: a compiled object is unchanged, so the links reproduce hashes XDR already knows.
- Repeats at each value: the cap forbids them.

## Review rounds

_No rounds yet._

## Question ledger

- Which command is "verify"? Self-pick: `lym gate test --wait --upto 2`, the push gate named in `CLAUDE.md`.
- Which knob replaces `go_parallel`? Self-pick: `tier_parallel`, per the gate config doc.
