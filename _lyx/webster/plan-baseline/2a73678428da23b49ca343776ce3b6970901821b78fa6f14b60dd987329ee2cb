# Card 12 — tier-tag-guards

**Edit:**
- `internal/planparser#namesGateOnlyTier`
- `internal/shellguard#rawGoCommandPattern`
- `internal/planparser_test#TestValidate_VerifyModuleWide`
- `internal/shellguard#TestRows_PatternsDenyAndAllow`
- `contracts/specs/loom-plan-spec.md`

**Intent:**
The guards keyed on the `tmux` and `llm` tags match `tier2` and `tier3` beside them (D5).

`namesGateOnlyTier`, the card Verify check behind `verify-module-wide`, reports a `-tags` value naming `tier2`, `tier3`, `tmux` or `llm`.
The shell guard's raw go row matches a `go test` whose `-tags` value names `tier2` or `tier3` as it does `tmux` and `llm`; its steer text keeps its wording.
`TestValidate_VerifyModuleWide` gains rows for `go test -tags tier2 ./internal/x` and `go test -tags tier3 ./internal/x` firing and `go vet -tags tier2` staying silent; `TestRows_PatternsDenyAndAllow` gains deny rows for `-tags tier2` and `-tags=tier3` and an allow row for `-tags tier1`.
`loom-plan-spec.md`'s `verify-module-wide` row names the tier spellings.

Bound: the guards still see static text only, as before.

**ImpactSummary:** Two guards refuse two more tag spellings; no refusal they already make changes.
