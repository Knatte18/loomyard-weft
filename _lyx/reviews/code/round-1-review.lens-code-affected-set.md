```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. Per change, the set I found and whether it is covered:

- **`|| integration` removed from guard files:** the guard files themselves are covered. `cmd/lyx/tiertags_test.go` is covered; it now refuses any old spelling. `impactset.FileTier`'s doc is covered. `pattern/PATTERN-test-speed.md:19` is NOT covered (finding 1).
- **Hub tier commands in `gate.yaml`:** covered. The board file already carries single tags (`tier1`, `tier2`, `tier3`). The `internal/gatecli/doc.go` example is covered.
- **`impactset` and `planglyph` package loads:** the code and the doc comments in `derive.go`, `guards.go`, `typesload.go`, `impactset/doc.go` and `planglyph/doc.go` are covered. `contracts/specs/loom-plan-spec.md:506` is NOT covered (finding 2).
- **`testtiming` (`defaultRedundancyTags`, spawn scan):** `main.go`, `redundancy.go` and `spawnscan.go` and their doc comments are covered. `docs/benchmarks/test-redundancy.md:3` names `integration,tmux`. It is a generated measurement report, so I did not flag it.
- **`tier2`/`tier3` refusal guards (`rows.go`, `verifymodule.go`):** both are covered, as are the `claudeengine` golden file and the planparser validate fixtures. The golden churn follows from the `rows.go` regex change.
- **Tier purity, tmux isolation and LLM tier guards:** covered. `knownTierTags` and `isolationTags` are tier-only, and `isTmuxTier` is rewritten.
- **`allow_tmux.go` constraint:** covered.
- **`glyphchain` `tagged.go`:** covered. The header and comment are respelled.
- **`PATTERN-test-isolation.md:14` (found by grep):** still describes the old tag sets.
- **`internal/testkit/lyxbin/lyxbin.go:1`:** still says "for `integration`, `tmux` and `llm` tests".

Findings:

1. **MEDIUM, `consistency`, `pattern/PATTERN-test-speed.md:19`.**
   - The line still says a guard file takes `tierN || integration` until the deployed `lyx` replaces the scan, and that the tier commands carry both spellings. The diff removed both, so the rule now describes a retired state.
   - It sits in the same `pattern/` file as the one-tag rule that `cmd/lyx/tiertags_test.go` enforces.
   - The decision record's "Comments outside the touched files keep their old spellings" only covers prose mentions in files the task does not touch. This is a live rule, and `CLAUDE.md` says a changed invariant goes into PATTERN in the same commit. Evidence: `pattern/PATTERN-test-speed.md:19`.

2. **LOW, `consistency`, `contracts/specs/loom-plan-spec.md:506`.**
   - It still says the type-resolving load uses the `tier0` through `tier3` tags "and the older `integration`, `tmux` and `llm` spellings". The load in `internal/planglyph/typesload.go` no longer passes them, and `planglyph/doc.go` was updated to match.
   - The decision record exempts only untouched files, and this is a spec stating current behavior. Evidence: `contracts/specs/loom-plan-spec.md:506` against `internal/planglyph/typesload.go:82`.

3. **LOW, `consistency`, `pattern/PATTERN-test-isolation.md:14`.**
   - It says the `TestMain` rule holds under "each of `integration`, `tmux` and `llm` alone, and the three together". No tier command or guard sets those tags any more.
   - The tmux isolation guard's `isolationTags` is now `tier0` through `tier3`. This is the one place that background file describes the guard's tag sets. Evidence: `pattern/PATTERN-test-isolation.md:14` against `cmd/lyx/tmuxisolation_test.go:32`.

4. **NIT, `consistency`, `internal/testkit/lyxbin/lyxbin.go:1`.** The package doc names `integration`, `tmux` and `llm` tests as the consumers. It is a file outside the diff, and the record's exemption plausibly covers it.

Tool calls:
1. A grep for transitional or old-spelling phrases, a read of `PATTERN-test-speed.md` lines 15–22, and a grep for the guard symbols. The manifest did not cover the docs or the guard symbols.
2. A grep for old-spelling phrases in non-test files, and a grep for `tier1,integration`, `tier2,tmux` and `tier3,llm`. The second confirmed no tier command or flag combo is left.
