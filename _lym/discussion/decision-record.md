## Goal

```markdown
{"error":"invalid json: invalid character 'h' looking for beginning of value","ok":false}

## Triage against main (2026-11-21)

- Red-first: d35d6556 removed the doc-file refusal, but `redFirstFailure` in `internal/loomshed/redfirstgate.go` still returns an ordinary failure for the unjudged and no-test exits, so each repeat spends a verify attempt; `GateResult.Pending` in `internal/shuttleengine/gate.go` is allowed only on PassOnCap or MayHold entries.
- Way forward: `internal/loomcli/darnwiring.go` always appends the config-or-model-spec clause, also to the stencil build-version refusal from `internal/stencilstore/skew.go`.
- Missing findings path: `internal/shedadapters/gateattempts.go` writes the file before the halt names it, so something removes it later; a suspect is the orphan scratch sweep in `internal/battencli/cleanup.go`, which empties a run's scratch when its records directory is missing. Reproduce before fixing.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-21): Scope after the corrupted board body

Decision: The board body was lost in a merge (orch error); only the triage section survived. Scope: (1) red-first: an unjudged or no-test refusal that only the parent can lift does not spend a verify attempt; the gate waits for the parent's decision (seen 2026-11-21 on gate-dotnet-fixes and red-first-overlay-non-go-files, which spent three attempts in two minutes while the parent's decision add was committing; issue #173). (2) issue #175: a stencil build-version refusal at Preflight carries only its own way forward, not the generic config-or-model-spec clause from internal/loomcli/darnwiring.go. (3) issue #174: a halt names a gate-findings path that no longer exists; reproduce what removes the file (suspect: the orphan scratch sweep in internal/battencli/cleanup.go) and fix that, or make the halt name a path that survives. Read issues #173, #174 and #175 with gh issue view for detail.

Rationale: The issues and the surviving triage section hold the lost notes' content.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - failed-hold-line-send-never-retried (class design, severity MEDIUM, open in rounds 1)
  - hold-send-failed-title-literal (class consistency, severity BLOCKING, open in rounds 1)
  - stale-findings-named-by-later-non-gate-stuck (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
