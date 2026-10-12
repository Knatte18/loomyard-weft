# Fixer-report — round 1

## Changes

- F1: `internal/loomcli/wiring.go` — the comment above `burlerengine.LoadConfig` no longer contrasts with the `(baseDir, module)` shape; it keeps only the optional-file reason.
- F2: `pattern/PATTERN-config-layers.md` and `internal/configengine/doc.go` — the rule now says no cleanup deletes a local override, and names the two removals: Seed-Child rewriting the pair's local layer, and reconcile deleting a file a prune emptied.
- F3: `internal/boardengine/runlock_test.go` — deleted `TestRunStatusProducer`, added the `RunStatusProducer` assertion to the "run status form" subtest; `internal/boardengine` row in `cmd/lyx/testdata/test-budget.yaml` lowered from 77 to 76.

All three fixed as proposed, one commit each.
A gate then flagged the F2 comment in `internal/configengine/doc.go` as fixed-column wrapped; it is now one sentence per line (Seed-Child removal, reconcile removal, "those are the only removals").
`lyx gate test ./internal/boardengine` and `lyx gate test ./cmd/lyx` pass.

## Disputed

None.

## Deferred

None.
