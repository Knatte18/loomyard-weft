# batcher-v2: measured start-context model and card-order rule

Both batches recorded `status: done`.

## Batch 01-batcher-start-model (cards 1–6, head `5479cfe91`)

The cost batchifier's start context became a base plus a growth per batch position, through the estimator, the dynamic program, the breakdown and the template.
The batchifier is told how many batches run before its cards, and webster's first init, validate and rebaseline pass that count.
`tools/tokencount` takes one repository flag for tags, plan commits and webster records, and the history flag is gone.
Each fork records its start context and its Merriam session.
Calibration fits the start coefficients by least squares and estimates every fork at its position, and it adds a per-batch cost table and a per-profile cost and quality summary.

Reported deviation: `cmd/lyx/testdata/test-budget.yaml`, where the `tools/tokencount` row rose from 7 to 10 because the cards add tests.

## Batch 07-card-order-rule (cards 7–11, head `72ae18c91`)

The plan spec, template and review rubric state that a plan runs strictly in card order.
The audit's location-free fabric-reference patterns became a rule callable on its own.
`planparser` gained a card-fabric-reference check over Verify values, the verify section, fenced blocks and inline spans, and both plan gates and validate-plan run it, with the spec row and the committed-data steer.
`accept-audit --batch` accepts a committed batch whose fabric-reference commands are read-only.

Reported deviation: `cmd/lyx/testdata/test-budget.yaml`, where the `internal/fabricengine`, `internal/planparser` and `internal/websterengine` rows each rose by one.

The fork noted two texts that still describe only the HEAD-at-start route: the `recover-batch` refusal way-forward in `internal/websterengine/recoverbatch.go` and the `accept-audit --batch` help line in `internal/webstercli/cli.go`.
Both stay correct but are more conservative than the new route, so rewording them is a follow-up.
