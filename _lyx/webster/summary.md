# sibling-overlap-check: engine siblings

Batch 01 (`engine-siblings`) was the only batch in the card list; it recorded `status: done` at head `69ad56e1be621d49b456b74561eb0f213f721feb`.

The fork's reply says it built `Sibling`, `SharedModuleLabels` and the sibling computation in `internal/boardengine/siblings.go`, and that `SetDeps`, `UpsertTasksBatch` and `MergeTasks` now return siblings.
It also reports implementing card 02 (`cli-sibling-notice`) in a second commit, though that card was not in this batch's list; webster recorded no batch for it.

Deviations from the digest: `internal/boardengine/siblings_test.go`, the one file the scope guard found changed outside the batch's own targets.
That is informational only.

## Verify gate fixes

The gate's first attempt failed on `cmd/lyx.TestCLIForm_EveryCommand`: the `lyx board upsert` `Long` help was 2143 bytes against the 2000 cap.
The fixer's reply says it shortened the sibling-notice sentence in `internal/boardcli/cli.go` for `upsert`, `upsert-batch`, `merge` and `set-deps`, and that `lyx gate test --tags integration ./cmd/lyx ./internal/boardcli` passes.
Its commits:

- `0784def29` — `fix: shorten the sibling notice help line so board upsert's Long fits the cap` (still 2029 bytes)
- `834789d22` — `fix: trim the sibling notice help line further so board upsert's Long meets the cap`
