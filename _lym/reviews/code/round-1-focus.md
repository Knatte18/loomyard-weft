---
round: 1
focus:
  - "The decision record's sections after `## Goal` are empty, so the measuring stick is the board entry alone: check each design point (command with no argument, slugs or date range, one darn `docs` task from a stencil, exact read cap, question set, stamped report under `docs/research/retro/`, operator picks proposals, the run files no board notes) against `internal/retrocli/cli.go` and `internal/retroengine/retro.go`, and flag any design point left unimplemented or any behavior the entry does not license."
  - "`contracts/stencils/retro/retro-template-task.md`: does the stencil carry the figures-first then LLM-reading flow, the routing rule (mechanical fault to a check, judgement call to a reviewer rubric line, no-effect instruction to a deletion, never a new prompt sentence by default), most-severe-first ordering, and tracing each proposal to a moment in a named run? Does `contracts/stencils/stencils.go` render every placeholder the stencil uses?"
  - "How `lym retro` finds the last retro (`internal/retroengine/retro.go`): check the stamp parsing, the tie and empty cases (no earlier report, no landed runs), the cap on runs, and the failure paths when the board or the report directory cannot be read."
  - "`tools/retro/retro.sh`, `facts.awk`, `figures.jq`, `transcript.jq`: check the shell for unquoted expansions, missing-input handling (a run with no transcript or no archive tag), and whether each figure the entry names (turns and cost per producer against medians, serial single lookups, guard refusals, verify-gate failures and fixer rounds, review rounds and recurring finding kinds) is produced."
  - "`internal/reportcli` changes (`PrimeGeometry` export): confirm the rename leaves no reference to the old name and that behavior is unchanged."
  - "Wiring and repository rules: `cmd/lym/main.go` registration, `cmd/lym/sandbox_coverage_test.go` allowlist, and `docs/overview.md`; check against the repository's constraints document that the docs the change kind requires land in the same diff."
  - "Test economy and coverage: `internal/retrocli/cli_test.go`, `internal/retroengine/retro_test.go`, `tools/retro/retro_integration_test.go`. Judge each against the economy bullets by name; check for any added behavior with no test (date-range and slug selection, the cap, release step failing)."
---

Seed focus only; each entry names where to look and leaves the verdict to the reviewer.
