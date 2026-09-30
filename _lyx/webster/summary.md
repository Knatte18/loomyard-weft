# fabric remove refuses before it archives

Batch 01 (refuse-before-archive) finished `done` at head `015c710c2`.
`Topology.Remove` now runs both no-force dirtiness checks and the status probes before pushing the `archive/<slug>/<tip>` tag, and pushes the tag before any portal or launcher removal.
`nameStrandedPortalTeardown` and its reconcile suffix are gone; the refusal base texts and the `ErrPairSiblingDirty`/`ErrArchiveFailed` sentinels are unchanged.
The old-order tests were rewritten for the new order, and the comments and `doc.go` passage were updated.

`record-batch` reported informational scope-outside-plan warnings.
They cover test symbols in `mutation_record_integration_test.go`, `remove_archive_integration_test.go` and `remove_refusal_remedy_integration_test.go`, all rewritten as part of the card.

The integration suite (`go build ./...`, `go test ./...`, `go test -tags integration ./...`, and the grep for `nameStrandedPortalTeardown`) reported `status: OK`.
