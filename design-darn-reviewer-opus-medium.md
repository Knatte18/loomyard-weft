# Darn Code-Review reviewer template default to opus[medium]

`darn-reviewer-opus-medium` · Task · enhancement, profiles, darn · running · Drift-Await

## Decision (operator, 2026-11-15)

Darn's Code-Review reviewer is `opus[medium]`, not `opus[high]`.
Commit `588ae6c5` set `loom.code_reviewer_model: "opus[high]"` in every darn block of `internal/runprofile/template.yaml`.
The loomyard hub's own `profiles.yaml` already reads `opus[medium]`; the template still seeds `opus[high]` into every new hub.

## Change

- Set `opus[medium]` in every darn block of `internal/runprofile/template.yaml`.
- Update `internal/runprofile/doc.go` and `internal/runprofile/runprofile_test.go`, which name `opus[high]`, and any doc outside `docs/research/` that does.
- Leave research reports as they are: they record past runs.
- Run `lym gate test ./internal/runprofile`.
