# Round 2 fixer report — discussion

## Changes

- F1 (MEDIUM), fixed as proposed.
  Checked against `rawGoCommandPattern` in `internal/shuttleengine/claudeengine/settings.go`, which denies only module-wide `go test|build|vet`, `tmux`/`llm`-tier `go test` and the lyx-owned variables, and against `scribe:golang-build` (`goimports -w`, `golangci-lint run`, `go install …@latest`) and `scribe:csharp-build`.
  `decision-record.md` D7's `Bound:` now states that the deny refuses only those go commands, that every other build-skill command runs unrefused outside the gate slots, and that the template comment and the operator's per-hub choice bound the hatch while the verify gate still runs the hub's build.
  The second `## Open risks` bullet now says such an operator sends agents to run that skill's commands, which the session settings mostly let through, outside the gate slots.
- F2 (NIT), fixed as proposed.
  `mergeresolve.New` is called from `internal/landingshed/mergein.go` and `internal/landingshed/finalize.go`.
  D4's first sentence now says the wiring that builds a code role's engine or its deps loads the `language` module once per run invocation; the second sentence keeps that `landingshed`'s deps carry the list from its wiring, without calling `landingshed` the loading constructor.

`support-log.md` is unchanged.

## Disputed

None.

## Deferred

None.
