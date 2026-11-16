# Card 20 — renumber-reads-active

**Create:**
- `plan:internal/loomcli#renumberDepsFor` -> `func renumberDepsFor(anchor string, active weaverengine.ActiveDeps) renumberDeps`
- `plan:internal/loomcli#errRenumberSeam` -> `var errRenumberSeam = errors.New("the run's active implementer cannot be read")`
- `plan:internal/loomcli#TestRenumberDepsFor` -> `func TestRenumberDepsFor(t *testing.T)`

**Edit:**
- `internal/loomcli#renumberDeps`
- `internal/loomcli#loomCLI.newRenumberDeps`
- `internal/loomcli#runCardVerb`
- `internal/loomcli#restoreRenumberVerb`
- `internal/loomcli#begunBoundError`
- `internal/loomcli#renumberWaysForward`
- `internal/loomcli#loomCLI.insertCardCmd`
- `internal/loomcli#loomCLI.removeCardCmd`
- `internal/loomcli#TestRenumberVerbs_Refusals`
- `internal/loomcli#TestRenumberVerbs`
- `//docs/overview.md`

**Extends:**
- `internal/loomcli/doc.go`

**Uses:**
- `plan:internal/weaverengine#Active`
- `plan:internal/weaverengine#RunDirs`
- `plan:internal/runplan#LastBegunCard`
- `internal/runplan#AcquireStateMutation`

**Intent:** The renumber verbs bound at the active implementer's last begun card, under its state-mutation lease (D3, D13).

- `renumberDepsFor` builds the verbs' side effects over an anchor and the seam's deps, and `newRenumberDeps` calls it with the loom CLI's anchor and seam.
  `loadState` returns `*runplan.State`, read through `weaverengine.Active`; a seam refusal is wrapped in `errRenumberSeam` with its text, never in the raw-I/O refusal.
  `acquireLease` takes the active implementer's lease (`runplan.AcquireStateMutation` over `RunDirs`' scratch directory), the lease Weaver records a batch begin under, so a renumber racing a begin is serialized.
  The active implementer is resolved once before the lease is taken; a seam refusal there is `errRenumberSeam` too.
- `renumberWaysForward` gains the `errRenumberSeam` row, whose step is the seam's own way forward, already in the refusal's text, followed by re-running the verb.
- `runCardVerb` bounds with `runplan.LastBegunCard`; `begunBoundError`'s text says "the last card the implementer has begun".
- `renumberDeps.acquireLease`'s comment, the `Long` help of `insert-card` and `remove-card`, `loomcli/doc.go`'s renumber paragraph and `docs/overview.md`'s renumber lines name the active implementer's lease and bound in place of webster's.

`TestRenumberDepsFor` is tier0 over temp anchors: with a planted Weaver state its `loadState` returns that state's begun bound and its lease file sits under `.lym/weaver/`; with a planted Webster state, under `.lym/webster/`; with both begun, `loadState` and the lease each refuse wrapping `errRenumberSeam`.
`TestRenumberVerbs_Refusals` gains one row: an `errRenumberSeam` load refusal names the seam's way forward and no transient re-run; the existing rows read the new bound text.

**ImpactSummary:** One loomcli verb family's state read and lease move to the seam, with its help, doc and overview text; the refusal table gains one row.
