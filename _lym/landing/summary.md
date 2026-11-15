# Darn's Code-Review reviewer defaults to opus[medium] in the template

Commit `588ae6c5` seeded `loom.code_reviewer_model: "opus[high]"` into every darn block of `internal/runprofile/template.yaml`.
The loomyard hub's own `profiles.yaml` already reads `opus[medium]`, so every new hub started on a more expensive reviewer than the operator runs.
Every darn block of the template, `docs`'s included, now seeds `opus[medium]`.

`internal/runprofile/doc.go` and `runprofile_test.go` name the new value.
The fixture strings in `internal/configsync/configsync_test.go`, `internal/hubreconcile/ensure_integration_test.go` and `internal/configcli/configcli_integration_test.go` mirror the template's darn lines for string replacement, so they carry `opus[medium]` too.

Research reports under `docs/research/` still say `opus[high]`: they record past runs.
Other `opus[high]` defaults, such as the arbiter, sampler and landing fixer, are separate settings and stay as they are.

## Design decisions

None beyond the operator's decision; no deviation from the board entry.
