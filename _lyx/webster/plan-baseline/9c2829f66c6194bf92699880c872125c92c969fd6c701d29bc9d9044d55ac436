# Card 14 — batten-innerrun-wiring

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli#childSpawnError`
- `internal/battencli#TestChildSpawnError`

**Create:**
- `internal/battencli#childRefusalOwner`
- `internal/battencli#ownerVerdictOf`
- `internal/battencli#childRestepped`
- `internal/battencli#TestOwnerVerdictOf`
- `internal/battencli#TestChildRestepped`

**Uses:**
- `internal/battencli#reconcileRun`
- `internal/battencli#childRefusalKind`
- `internal/battenshed#ChildOwnedElsewhereError`
- `internal/battenshed#OwnerVerdict`
- `internal/shedrun#StartOwnedElsewhereKind`
- `internal/fabricengine#ParentHoldsTaskWork`
- `internal/fabricengine#EntryOwnedElsewhere`
- `internal/tendercli#PauseOwnedElsewhere`

**Intent:**
Batten wires card 12's seams for a live run.
`ownerVerdictOf` maps a run-scope report onto `battenshed.OwnerVerdict`: `Unreachable` from the report, `Elsewhere` and `Owner` from the slug's owned_elsewhere entry, owned here otherwise.
`wire`'s `InnerRun` fills `OwnerCheck` with a judge-only run-scope reconcile of the slug from the prime over `pairteardown`'s hub seams, mapped by `ownerVerdictOf`; `PauseOwned` with `tendercli.PauseOwnedElsewhere`; `ParentHoldsTaskWork` with `fabricengine.ParentHoldsTaskWork` over the prime and the located task worktree; and `Restepped` with `childRestepped` over the child's settle records (`sheddriver.ReadSettleRecords`).
`childRestepped` reports whether a record's stop key equals the status's stop key (`sheddriver.StopKeyOf`) and its action is `shedrun.SettleRestep`.
`childSpawnError` wraps a child output whose refusal kind is `owned_elsewhere` in `*battenshed.ChildOwnedElsewhereError` carrying `childRefusalOwner`, which reads the `owner` field of the first refusal envelope line carrying one, so InnerRun pauses instead of failing the spawn.
`TestChildSpawnError` gains a row: an `owned_elsewhere` envelope with an owner yields an error `errors.As` reads as `ChildOwnedElsewhereError` with that owner.
`TestOwnerVerdictOf` covers an owned-elsewhere entry, an active entry, an unreachable report and a report without the slug.
`TestChildRestepped` covers a restep at the same stop, a restep at an earlier stop and a different action at the same stop.

**ImpactSummary:** battencli's InnerRun wiring and spawn error mapping; the live owner check of every batten run turns on with this card.

**Commit:** 14: batten: wire the live owner check, its pause and the Finalize carve-out
