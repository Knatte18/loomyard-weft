# Card 11 — batten-innerrun-wiring

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli#childSpawnError`
- `internal/battencli#TestChildSpawnError`

**Create:**
- `plan:internal/battencli#childRefusalOwner` -> `func childRefusalOwner(childOutput string) string`
- `plan:internal/battencli#ownerVerdictOf` -> `func ownerVerdictOf(report fabricengine.OriginReport, slug string) battenshed.OwnerVerdict`
- `plan:internal/battencli#childRestepped` -> `func childRestepped(records []shedrun.SettleRecord, status shedengine.Status) bool`
- `plan:internal/tendercli#PauseOwnedElsewhere` -> `func PauseOwnedElsewhere(location *lyxcwd.Location, slug string) error`
- `plan:internal/battencli#TestOwnerVerdictOf` -> `func TestOwnerVerdictOf(t *testing.T)`
- `plan:internal/battencli#TestChildRestepped` -> `func TestChildRestepped(t *testing.T)`

**Uses:**
- `internal/battencli#reconcileRun`
- `internal/battencli#childRefusalKind`
- `plan:internal/battenshed#ChildOwnedElsewhereError`
- `plan:internal/battenshed#OwnerVerdict`
- `plan:internal/shedrun#StartOwnedElsewhereKind`
- `plan:internal/fabricengine#ParentHoldsTaskWork`
- `plan:internal/fabricengine#EntryOwnedElsewhere`
- `plan:internal/pauserecord#OwnerCaller`
- `internal/tenderengine#Pause`

**Intent:**
Batten wires card 10's seams for a live run.
`ownerVerdictOf` maps a run-scope report onto `battenshed.OwnerVerdict`: `Unreachable` from the report, `Elsewhere` and `Owner` from the slug's owned_elsewhere entry, owned here otherwise.
`wire`'s `InnerRun` fills `OwnerCheck` with a judge-only run-scope reconcile of the slug from the prime over `pairteardown`'s hub seams, mapped by `ownerVerdictOf`; `PauseOwned` with `tendercli.PauseOwnedElsewhere`; `ParentHoldsTaskWork` with `fabricengine.ParentHoldsTaskWork` over the prime and the located task worktree; and `Restepped` with `childRestepped` over the child's settle records (`sheddriver.ReadSettleRecords`).
`tendercli.PauseOwnedElsewhere`, the one owner pause every caller uses, builds tender's hub deps with `HubDeps` and pauses the slug through `tenderengine.Pause` with a ten second grace and `Caller` `pauserecord.OwnerCaller`, so tender interrupts the child's running turns and a slug entry already standing adds none.
`childRestepped` reports whether a record's stop key equals the status's stop key (`sheddriver.StopKeyOf`) and its action is `shedrun.SettleRestep`.
`childSpawnError` wraps a child output whose refusal kind is `owned_elsewhere` in `*battenshed.ChildOwnedElsewhereError` carrying `childRefusalOwner`, which reads the `owner` field of the first refusal envelope line carrying one, so InnerRun pauses instead of failing the spawn.
`TestChildSpawnError` gains rows: an `owned_elsewhere` envelope with an owner yields an error `errors.As` reads as `ChildOwnedElsewhereError` with that owner, and one with no `owner` field yields it with an empty owner, which the notice words as "another machine".
`TestOwnerVerdictOf` covers an owned-elsewhere entry, an active entry, an unreachable report and a report without the slug.
`TestChildRestepped` covers a restep at the same stop, a restep at an earlier stop and a different action at the same stop.

**ImpactSummary:** battencli's InnerRun wiring and spawn error mapping; the live owner check of every batten run turns on with this card.

**Commit:** 11: batten: wire the live owner check, its pause and the Finalize carve-out
