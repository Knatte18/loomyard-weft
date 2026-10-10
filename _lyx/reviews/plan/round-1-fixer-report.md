# Plan-Review round 1 — fixer report

Every finding was validated against the code and the decision record and fixed.
`lyx loom validate-plan` passes the edited plan (`ok: true`, informational `create-new-unit` findings only).

## Fixed

- **F1** (card-verify and plan-gate waits invisible in status).
  Card 1's `Wait` record now carries a site (the `lyx gate test` command or a Go-side site label), and `WaitDir` is documented as the record directory for every waiter with no `verifytree` marker.
  Card 6: `rerunCardVerifies` takes a `waitDir` (re-sign arrow updated), writes a `WriteWait` record for `card verify NN-<slug>` before acquiring and removes it once the acquire returns; `RecordBatch` passes `gateslot.WaitDir(deps.Geom.AnchorRoot)`, the directory status reads; the new test row asserts no record remains.
  Card 9: `NewSlottedIndex` takes a `waitDir` (arrow updated), `goListLoader.load` records its `plan gate caller-uncovered` wait the same way, and each wiring site passes `gateslot.WaitDir` over its anchor; the integration row asserts the record is present while waiting and gone after.
  Both cards gained `plan:internal/gateslot#WaitDir` and `#WriteWait` under `Uses:`, both created by card 1.
  Card 11: `gateWaiting` reports every live wait record as `<producer>: <site> waiting for a gate slot …`, and the status spec names the sites that write it.
- **F2** (inheritance edge without a bound).
  Card 10's Intent states D4's and D7's bound on a forged `gateslot.InheritEnv`: it runs unslotted inside that holder's slot, card 12's deny refuses an agent command naming it, and an operator shell or a script hiding the name is outside the bound; `doc.go` documents "the bounds above", which now include it.
- **F3** (budget rows full).
  Confirmed: `internal/websterengine` holds 192 top-level tests against its 192 row, and `internal/loomshed` 69 against 69.
  Cards 15 and 22 list `cmd/lyx/testdata/test-budget.yaml` under `Edit:`, state their new table-driven test (`renderBatchGate`, `PublishFailureNote`) and the row's rise by one.
- **F4** (stale package docs).
  Card 14 edits `internal/websterengine/doc.go`, rewriting the card-gate paragraph to D1.
  Card 15 also edits it, rewriting the "no batch-wide verify" sentence to D2's batch gate in the card that adds the gate.
  Card 21 edits `internal/loomshed/doc.go`, rewriting "compiles the `tmux` and `llm` tiers and never runs them" to the record-driven `tmux` pass.
- **F5** (text-only push line left to a merge).
  Confirmed `0cdd14ac9` is not an ancestor of HEAD.
  Card 16 now writes main's line itself, verbatim and indented under the rewritten bullet, so Publish's merge-in meets identical text.
- **F6** (empty impacted set).
  Card 21 states that an empty `Packages` with no fallback skips the impacted-set `tmux` pass and keeps the named-test steps, and `publishfailure_test.go`'s command table gains the empty-package-list row for `publish_verify`.

## Disputed

None.

## Deferred

None.
