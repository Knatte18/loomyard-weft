```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that each set-shaped finding's fix covers every member of its set, not only the members the review named.

Unchecked findings: none. F1, F3, F4, F5, F10, F11 and F12 have set-shaped premises, and each was rerun. F2, F6, F7, F8 and F9 have no set premise.

- **F1:** card 9 now skips every file target not ending in `.go`, so no other non-Go kind is left uncovered.
- **F3:** cards 8 and 9 both drop their done-card and `ValidateDispatch` rows and keep their `language: csharp` rows. Card 9 drops all four loomcli test targets. Card 8 drops three, keeping `TestGateParity_PlanGate` and `TestPlanWriteEntry_GateConfig`.
- **F4:** the only new test file in the plan is `citations_integration_test.go`, now a `**Create:**` path. Every other test the plan adds goes into an existing test file, including card 4's new `TestTrackedRootEntries` in `worktree_test.go`.
- **F5:** both `_test` pseudo-unit refs that card 8 expects to pass now also assert `glyph-not-found`: `internal/c_test#TestDeps` and `internal/c_test#`. The only other `_test` ref in cards 7–9 is card 9's Create handle `plan:internal/d_test#TestNew`, which the gate does not resolve.
- **F10:** the consumer list exists only once, in `internal/planglyph/repo.go:51`, and card 3 now edits `resolveTargets` there.
- **F11:** the stale field list sits at `internal/loomcli/wiring.go:259`. `validatePlanCmd` already reads `PlanIndex` (`wiring.go:309` fills it), so card 4's sentence adding `PlanIndex` covers both verbs the comment names.
- **F12:** every `ResolveGlyphs` caller guards against an empty list: card 4 calls it "only when there is at least one", card 8 "only when the plan has cards and some marker exists", card 9 "only when there is one".

no findings

Tool calls:
1. Grepped `internal/planglyph` for the Resolve-consumer list and cards 7–9 for `_test#` refs, because my context did not show all occurrences of either.
2. Grepped `internal/loomcli/wiring.go` for the field names the comment lists and the plan cards for `ResolveGlyphs` callers. My context showed only part of the comment and did not show the full set of callers.
3. Grepped the plan cards for new files they create, because my context did not show the full set.
