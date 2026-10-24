```yaml
lens: code-pattern-fit
outcome: reported
```

**Lens: code-pattern-fit. Each rule I checked, held or broken:**

- `PATTERN-sole-parsers` — held. `CardNamesPatternEntry` lives in `internal/planparser` and `cardHeadroomFindings` reads card text only through it and `CardPatternEntries`. The `Card.Text` comment update in `plan.go` matches the new reader.
- `PATTERN-refusal-way-forward` — held. Each `pattern-card-headroom` finding appends `patternHeadroomWayForward`. The test rows assert `wantDetail` and `Card` on every finding.
- `PATTERN-test-speed` — held. The new coverage is rows in the existing `TestNewPlanGate` table plus one table-driven `TestCardNamesPatternEntry`, both with `t.Parallel`. The `tier0` tag in `patternentry_test.go` is unchanged, and no sleeps or real clocks were added.
- `PATTERN-gate-slot`, `PATTERN-config-layers`, `PATTERN-leaf-packages` and the other entries — not touched. The diff adds no `PATTERN.md` or `pattern/` change, and the commit declares none.
- "Declared PATTERN changes landed" — nothing to check. I saw no declared-changes list, and the decision record declares no PATTERN changes.
- "Commit messages and departures" — held. The single commit has no `Pattern-Departure:` trailer and none is needed. It ends with `Co-Authored-By: Claude Opus 5.5` where the session attribution says Sonnet 5.5, which is not a PATTERN matter and I raise no finding.
- `CLAUDE.md` "Docs land in the same commit" — held. `doc.go` and `contracts/specs/loom-plan-spec.md` are both updated in the same commit.
- `CLAUDE.md` "Markdown: semantic line breaks" — held. The new lines in the spec and the stencil are one sentence per line.
- The Go comment rules on the new symbols — held. `cardHeadroomFindings`, `CardNamesPatternEntry` and the new constants have doc comments that start with the symbol name.
- Stencil wording (`loom-template-plan.md`) — the two added lines state the new gate behavior and drop nothing. The rubric's other half, whether the answer key licenses the stencil change at all, is the handler's.

**Findings:** none.

**Tool calls:**
1. `cat pattern/PATTERN-sole-parsers.md` and a grep for `Pattern-Departure` in `PATTERN.md`: the manifest did not hold the background file text.
