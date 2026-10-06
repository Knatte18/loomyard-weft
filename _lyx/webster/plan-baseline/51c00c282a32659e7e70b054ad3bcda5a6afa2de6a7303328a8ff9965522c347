# Card 15 — shuttle-prune

**Edit:**
- `internal/shuttlecli#`
- `internal/shuttleengine/claudeengine#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of `internal/shuttlecli` and `internal/shuttleengine/claudeengine`.
`internal/shuttleengine/claudeengine` is a separate package from core's `internal/shuttleengine`, whose own files stay untouched (Decision 1).

`internal/shuttleengine/claudeengine`'s candidates are fold leads over its provider-specific parsing (`PATTERN-shuttle-provider-seam`), and its `envfile_integration_test.go` is a tier-move lead.
The skill-loading carve-out of `ownership-boundary` applies here: `TestSkillLoadSequence`, `TestSkillUnknown` and every other test in the package naming `Spec.Skills`, `DefaultSkillLoadTimeout`, `loadSkill` or `loadSkillsThenPrompt`, with the `SkillLoader` fakes they use, are left as they are, and a candidate among them is kept with `//testtiming:keep` naming `skills-in-one-turn`.
`internal/shuttlecli` has `llm` files that Decision 6's helper rule covers.

**ImpactSummary:** Test files of two shuttle packages only; no production change, and core's `internal/shuttleengine` files are untouched.

**Commit:** 15: Prune shuttle CLI and provider subpackage tests

**Verify:**
go vet -tags llm ./internal/shuttlecli/
