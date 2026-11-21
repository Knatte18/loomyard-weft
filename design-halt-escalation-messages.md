# Parent-only refusals stop retrying; halt messages name real files and ways

`halt-escalation-messages` · Task · bug, gate, loom, stencil, shed · running · Drift-Recheck

{"error":"invalid json: invalid character 'h' looking for beginning of value","ok":false}

## Triage against main (2026-11-21)

- Red-first: d35d6556 removed the doc-file refusal, but `redFirstFailure` in `internal/loomshed/redfirstgate.go` still returns an ordinary failure for the unjudged and no-test exits, so each repeat spends a verify attempt; `GateResult.Pending` in `internal/shuttleengine/gate.go` is allowed only on PassOnCap or MayHold entries.
- Way forward: `internal/loomcli/darnwiring.go` always appends the config-or-model-spec clause, also to the stencil build-version refusal from `internal/stencilstore/skew.go`.
- Missing findings path: `internal/shedadapters/gateattempts.go` writes the file before the halt names it, so something removes it later; a suspect is the orphan scratch sweep in `internal/battencli/cleanup.go`, which empties a run's scratch when its records directory is missing. Reproduce before fixing.
