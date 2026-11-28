# A gate run under the parent lock takes the next free gate slot ahead of runs that hold no lock

`lock-holder-gates-first` · Task · enhancement, gate, landing · starting · Seed-Child

## Seen (2026-11-28)

A run gating tiers 0 to 2 under the parent lock waited for the same four gate slots as the runs queued behind its lock,
whose lock-free gates its landing would make stale anyway.
`docs/research/webster/webster-time-2026-11-08.md` (lever 4) also measured 3.1 min of slot wait per Webster row, 27 min in one.

## Idea

Give gate slot requests a priority: a gate run under the parent lock first, then implementer gates, then the rest.
