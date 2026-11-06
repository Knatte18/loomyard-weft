```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. I found no findings.

**Set per change**

1. **`--pattern-action withdraws` and `pattern_action` in a `--from` file** (flag text, help, refusals, `pattern_withdrawn` key). All places that name the flag or key:
   - The CLI help text, flag descriptions and way-forward constants in `decision.go` (lines 98, 109–110, 1280–1306): covered. The unknown-key allow-list at `decision.go:247` needs no change, since withdrawal reuses the existing keys.
   - `arbiterprompt.go:20`, `orch-role.md:236`, `docs/overview.md:596`, `loom-discussion-spec.md:42` and `loomcli/doc.go:345`: all updated.
   - `patternplan.go:26` mentions `decision add --pattern-action changes` only as a headroom fix, so it needs no edit.
   - No other contract, stencil or doc names the flag.
2. **`AddedDecision.WithdrawnPattern`, `ErrPatternBulletAbsent`, the `Withdraws:` line and the `CheckAddedDecisionsKept` exemption**:
   - `discussionparser/doc.go`, the `loom-discussion-spec.md` rule at line 31 and the `CheckAddedDecisionsKept` doc comment: all updated.
   - The other readers of added entries (`AddedDecisionTexts`, `IdentityText` and `SplicePart`) treat the extra `Withdraws:` line as part of the entry's span, so they are unaffected.
   - The decision-add `--from` batch and `decisionEntries` duplicate-ID check are covered by `patternFormBroken`.
3. **Raddle `pattern_clusters`, `PatternEntryIDs`, `MapPatternChanges` and the three new checks**:
   - `Part.PatternChanges` is read only in `check.go`, `raddlefile.go` (now through `PatternEntryIDs`) and `SplicePart`, which still gets the Sketch's own ids for the Sketch's record.
   - `raddle-proposal-spec.md` documents the key and all three checks, and the raddle stencil gets the mapping rule.
   - The spec's JSON example at line 126 omits the optional key, which is fine.
   - `raddleparser/doc.go` is updated.
   - The sole-parsers import rule holds: `check.go` adds only the `discussionparser` import, and `proposal.go` adds stdlib `slices`.
4. **`BoardContext.LivePattern`**:
   - Production sets it only in `loomcli/raddlecontext.go:62`.
   - `loomshed/raddlefile.go:379` builds a `BoardContext` without it, but that context goes to `RecheckBoard`, which never reaches the cluster check at `check.go:220`.
   - The test contexts at `raddlecheck_test.go:73` and `raddleverbs_test.go:51` carry no clusters, so they never call it.
   - `raddleparser/fixture_test.go` sets it. Covered.
5. **Declared PATTERN changes and commit messages**: the constraints section declares no PATTERN changes, so there is nothing to land. No commit carries a `Pattern-Departure:` trailer, and the third commit's help-text rewording stays within `decision.go` help.

**Tool calls (6 allowed, 5 used)**
1. grep for `pattern-action` and `pattern_action` outside tests: to find every doc, stencil and prompt naming the flag.
2. grep for `.PatternChanges` and `pattern_changes` in raddle code and docs: to find other readers of the part field.
3. grep for `LivePattern` and `BoardContext{` constructions: to find other `BoardContext` builders.
4. Read `loomshed/raddlefile.go:355–405`: the filing-time `BoardContext` construction was outside my context.
5. grep of `check.go`, `recheck.go`, the spec and the stencils for `LivePattern`, `Check(` and `pattern_clusters`: to confirm `RecheckBoard` does not run the cluster checks.
