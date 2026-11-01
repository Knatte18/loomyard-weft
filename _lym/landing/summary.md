# Make wide the batcher template's active profile

`internal/batcher/template.yaml` now ships `active: "wide"`, so every hub without its own `batcher.yaml` override groups cards under the 300000-token budget instead of 200000.
`wide` is the `cautious` cost batchifier with `budget: 300000`, the profile this hub has run since 2026-10-22.
`docs/research/batch-budget-trial-2026-11-01/report.md` is the evidence: over 23 `wide` runs it cost no more per card than `cautious` and showed no quality loss.

## Changes

- The template gains `wide` as its first cost profile and points `active:` at it.
  `cautious` (budget 200000) and `identity` stay as profiles a hub may choose.
  `cautious` repeats `wide`'s estimator weights, so the two differ only by budget.
- No estimator weight changes.
- Comments and docs that named `cautious` as the template default now name `wide`: `internal/batcher` (`config.go`, `registry.go`, `doc.go`), `internal/websterengine/doc.go`, `docs/overview.md` and `tools/sandbox/SANDBOX-WEBSTER-SUITE.md`.
- `internal/configsync`: a hub whose `batcher.yaml` still carries the former template value `active: "cautious"` has it stripped on sync, as it already did for the old `profiles` value, so that hub follows the new default instead of staying pinned to the old one.

## Design decisions

- `cautious` is kept whole rather than defined as an override of `wide`, because profiles merge by name key by key and a hub can tune either one independently.
- A hub that deliberately chose `cautious` and only ever matched the template value cannot be told apart from one that never set it; the strip moves both to `wide`.
  That hub can set `active: cautious` again.

## Tests

- `internal/batcher/config_test.go` resolves `wide` from the template, an absent `batcher.yaml` and an absent `_lym/`, and pins that the template's `wide` and `cautious` share weights and differ only in budget.
- `internal/configsync/configsync_test.go` covers the former-default strip of `active: "cautious"`.
- `internal/webstercli/verbs_test.go` follows the new default name.

## Check by hand

- The former-default strip of `active: "cautious"` in `internal/configsync/configsync.go`: confirm `configsync_test.go` pins both the strip of the former value and the keeping of a differing `active:`.
- Run `lym loom gate test ./internal/batcher ./internal/configsync ./internal/webstercli`; no run result is recorded for this change.
