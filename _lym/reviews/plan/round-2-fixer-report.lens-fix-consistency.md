```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: does the change set agree with its siblings and cross-references, and did any change leave a twin elsewhere unchanged?

- **LOW**, card 6 (`06-layout-mutation-record.md`), the added bullet that rewrites the `Snapshot`/`Extend` calls on a record value as `fabriclayout.MutationsOf`.
  The card's **ImpactSummary** still says "The recorder keeps its name and call shape, so its call sites compile unchanged".
  The new bullet now edits three call sites in the engine and `internal/pairteardown`, so the summary leaves them out.
  Evidence: these are the only production `Extend` calls on a record value: `internal/fabricengine/originreconcile.go:815`, `internal/fabricengine/originreconcileended.go:125` and `internal/pairteardown/teardown.go:584`. Every other `.Extend(` caller is a `rec *Mutations` recorder (`fabriccli/*.go`, `destroy.go:2255`, `unwire.go:76`, `hubpull.go:148`). `anchorchange.go:387` calls `Snapshot()` on `recovered *Mutations` (`anchorchange.go:47`), a recorder. So the bullet covers every site, but the summary sentence is now partly stale.

Nothing else: the overview's new figures (about 700 deleted, about 180 lines down) appear nowhere else in the plan. Card 15's references to the decision "the foundation's tests shrink" quote no numbers. Card 14's rule that a phase enters `Phases` only after a successful read, and its "latest phase" definition, agree with card 15's fixed store order and per-field `Store`.

Tool calls:
1. grep for `Extend`/`Snapshot` on record values in production code, to check the card 6 bullet covers every site; my context held only the three sites the review named.
2. grep for every production `.Extend(` caller, to check no other caller depends on a record value.
3. grep for the type of `anchorchange.go`'s `recovered`, which call 2 showed calls `Snapshot()` and whose type my context lacked.
