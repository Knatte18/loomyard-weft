# Fanned reviewer gathers its fork pre-read in parallel waves

A fanned review round's handler reads the target before spawning its forks.
A cost audit found it did this one call a turn: 89% of the round-1 Plan reviewer's turns made a single call, and about 41% of its weight was turns that could have been merged into others.
The general batching sentence it already carried did not change that.

`contracts/stencils/burler/burler-fork-reading.md` now tells the handler how to batch the pre-read:

1. After reading the target, list every symbol, state, output and PATTERN entry it names, then issue one separate call for each in a single message: definition searches, caller/test/doc/fixture searches, cross-process path searches, writer/reader searches, `lym board get`, and the `pattern/` background reads.
2. In the next message, read together every definition and call site the first wave found, with the callees of each changed symbol and the failure, restart and resume paths of each change.
3. Add a further wave only for a hop the second wave opened, such as a spawned process or restart path.

A call moves to a later message only when its arguments come from an earlier result.
`internal/burlerengine/doc.go` records this in the description of the explore step's `fork_reading` marker.

The stencil renders in fanned rounds only, so solo rounds are unchanged.

## Design decisions

- The fix is a named wave structure in the stencil, not a heddle line or a model change.
  The task offered both of those as alternatives; this change trials only the stencil wording.
- Waves are described by what each one's calls are, so the handler can tell which calls are independent without being given a fixed call count.

## To check by hand

- The change is prompt text and was not measured. Rerun the scripts in `docs/research/review-turn-audit-2026-11-01` on a fanned round-1 Plan review and compare calls per turn and removable turns against the audit's 1.05 calls a turn.
- If the handler still gathers serially, the next trial is a heddle line or running round 1 on the model that batched.
