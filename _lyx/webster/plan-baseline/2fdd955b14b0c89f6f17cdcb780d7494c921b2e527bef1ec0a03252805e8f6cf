# Card 1 — Relaunch learns from Admit that its target is already in effect

**Edit:**
- `internal/shedengine#RelaunchRequest`
- `internal/shedengine#Relaunch`
- `internal/shedengine#TestRelaunch`
- `internal/shedengine/doc.go`
- `internal/loomcli#consumeArbiterAttempt`
- `internal/loomcli#consumeTacklerAttempt`

**Intent:**
D1's generic skip signal in `shedengine`, with no loom type named (`PATTERN-shed-producer-seam`).

`RelaunchRequest.Admit`'s field type changes from `func(cur Status) error` to `func(cur Status) (inEffect bool, err error)`.
Its doc comment says: an error is returned unchanged and nothing is written; `inEffect` true reports that the run already stands where `Target` would move it, so the re-launch performs no goto and writes the status unchanged.

`Relaunch` calls `Admit` exactly where it does today, after every check including `CheckGotoTarget`, and applies the `Target` move only when `Target` is non-empty and `inEffect` is false.
With `inEffect` true it writes the status unchanged, as a re-launch with no target does, so the history gains no entry.
Its doc comment, and the `Relaunch` paragraph of `internal/shedengine/doc.go`, gain one sentence saying so.

The two loom callers change only mechanically in this card: each `Admit` closure returns `false` beside its existing error or the applied-mark write's error, so their behaviour is unchanged until card 2.

`TestRelaunch`'s `admitErr` column is joined by an `admitInEffect bool` column that the row's `Admit` closure returns, and one row is added:
"target in effect writes the status unchanged": seed `seedWith(StateBlocked, "B")`, target `"A"`, `admitInEffect: true`, one admit; with no `check` the row lands in the existing "status file changed without a target" `reflect.DeepEqual(before, after)` branch, which fails if the goto is applied.
The existing "backward target keeps the pause conditions" row stays the proof that `inEffect` false still moves the run.

Bound: the signal skips only the goto of an already-admitted re-launch; every refusal `Relaunch` makes before `Admit` runs as before, and `inEffect` lets nothing through that `Admit` itself did not admit.

**ImpactSummary:** One struct field type and one function in `shedengine`; its only callers are the two loom consume closures, adjusted here with no behaviour change.

**Commit:** 1: Let Relaunch skip a goto Admit reports already in effect
