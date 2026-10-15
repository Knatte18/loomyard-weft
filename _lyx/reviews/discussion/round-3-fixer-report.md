# Fixer report — discussion round 3

## Changes

- **F1** (fixed as proposed): D7's Bound now cites `rawGoCommandPattern` in `internal/shuttleengine/claudeengine/settings.go` and states its three shapes: a `go test`, `go build` or `go vet` with a word containing `...` or the word `all`, a `go test` whose `-tags` names `tmux` or `llm`, and a command naming the gate-slot, strand-name or prebuilt-binary variable.
  It adds that the deny is a guardrail on static text, and the unrefused examples name a one-package `go test` such as `./internal/x`.
  The second Open risks bullet now points at D7's Bound instead of saying "mostly let through".
  Premise checked against `settings.go`, whose `rawGoCommandPattern` doc comment lists the same three shapes and the static-text limit.
- **F2** (fixed as proposed): D4's Decision now says the module loads once per process, that `landingshed` loads no config, and that `internal/loomcli`'s `wire()` loads the module beside `landing.yaml` and tells the list through `landingshed.Deps` to `mergeresolve.Deps`, keeping `landingDeps` free of I/O.
  The Rationale now reads "Loading once per process gives every spawn that process builds the same list."
  Notes for the plan writer gains the `landingDeps` call sites, `arm.go` and `sharedbootstrap.go`.
  Premise checked: `landingDeps`'s doc comment says it does no I/O, `wire()` loads `landing.yaml` in `internal/loomcli/wiring.go`, and `landingDeps` is called from `arm.go:429` and `sharedbootstrap.go:335`.

`support-log.md` is unchanged: the review places F2's fix in the record, which is the file the plan writer reads.

## Disputed

None.

## Deferred

None.
