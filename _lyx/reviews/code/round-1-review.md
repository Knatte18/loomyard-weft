---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: cmd/lyx/tiertags_test.go:73
    summary: firstRetiredTierSpelling uses Eval, which short-circuits &&, so a retired spelling right of a false term (tier1 && integration, linux && integration, !tier1 && tmux) is never seen and the scan passes the file
    fix: In firstRetiredTierSpelling replace the Eval closure with a walk over the constraint.Expr tree (handle *constraint.AndExpr, *constraint.OrExpr, *constraint.NotExpr and *constraint.TagExpr, descending into both operands) that returns the first TagExpr whose Tag is a key of retiredTierSpellings. Add TestTierTagFailure_FixtureSources rows with wantFail true for "//go:build tier1 && integration" and "//go:build linux && tmux" using testFunc.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-pattern-fit
    location: pattern/PATTERN-test-speed.md:19
    summary: The tier-tags rule still says a guard file takes tierN || integration until the deployed lyx replaces the guard scan, and that the hub tier commands carry both spellings; this diff ends that transition and the scan now refuses the term
    fix: Rewrite the bullet at line 19 to say a file holding a //lyx:guard test takes its one tierN tag like any other test file, and that cmd/lyx/tiertags_test.go refuses a constraint naming integration, tmux or llm in any test file. Drop the transition and both-spellings sentences.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-pattern-fit
    location: pattern/PATTERN-test-isolation.md:14
    summary: The background file still describes the TestMain rule under integration, tmux and llm tag sets and tells authors to tag a file tmux, a spelling the one-tag scan now refuses, while the isolation guard enforces tier0 through tier3 sets
    fix: In pattern/PATTERN-test-isolation.md respell line 14 to "untagged, each of tier0 through tier3 alone, and all four together", and respell the tmux, llm and integration mentions at lines 27, 28, 47 and 48 to tier2, tier3 and tier1 (line 48 tells the author to tag the file tier2). Read each line first and change only the tag spellings.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/loom-plan-spec.md:506
    summary: The spec says the type-resolving load passes the older integration, tmux and llm spellings, but internal/planglyph/typesload.go no longer does and planglyph/doc.go was already updated
    fix: In contracts/specs/loom-plan-spec.md line 506 delete "and the older `integration`, `tmux` and `llm` spellings" so the sentence names the tier0 through tier3 tags only.
---

### [MEDIUM] Retired-spelling scan misses a spelling behind a false && term

**Location:** cmd/lyx/tiertags_test.go:73
**Issue:** The board entry's decision says the one-tag scan fails any test file whose constraint names integration, tmux or llm. `firstRetiredTierSpelling` finds the spelling by evaluating the constraint with a callback that returns false for every tag, and `constraint.AndExpr.Eval` short-circuits, so the right operand of `tier1 && integration` is never queried. The file passes the scan, `admittedTiers` reads `integration` as a platform term, and the file runs under no tier command: the silent test loss the decision exists to prevent. The fixture rows cover `||`, leading and negated forms only.

### [MEDIUM] PATTERN-test-speed still describes the transitional guard tag

**Location:** pattern/PATTERN-test-speed.md:19
**Issue:** The bullet states a rule that no longer holds: a guard file with `|| integration` now fails `cmd/lyx/tiertags_test.go`, and the hub's tier commands carry one tag each. The diff touches no file under `pattern/` or `PATTERN.md`, though CLAUDE.md puts invariant changes into PATTERN in the same commit.

### [MEDIUM] PATTERN-test-isolation still names the retired tag sets

**Location:** pattern/PATTERN-test-isolation.md:14
**Issue:** Line 14 lists the tag sets the TestMain rule holds under as the old three spellings, which `isolationTags` and `isolationTagSets` in `cmd/lyx/tmuxisolation_test.go` no longer use. Lines 47 and 48 tell an author to tag a file `tmux`, which the one-tag scan now rejects.

### [LOW] loom-plan-spec still lists the older spellings for the types load

**Location:** contracts/specs/loom-plan-spec.md:506
**Issue:** The spec describes the `go/packages` load as using the older spellings besides tier0 through tier3. `goListLoader.load` passes `-tags=tier0,tier1,tier2,tier3` only, and the decision record's exemption covers prose in files the task does not touch for a different reason: this is a spec of current behavior for the same code the diff changes.

## Rejected

- Scope against the board entry: all eight bullets land (guard files and scan, hub `gate.yaml` already single-tagged on the board, impactset and planglyph loads, testtiming scan and defaults, the two refusal guards, purity and isolation guards, `allow_tmux.go`, `tagged.go`); no bullet is missing and the golden churn follows from the `rows.go` pattern change.
- lens:code-pattern-fit, `contracts/stencils/govalues_test.go:102` banned list keeps `-tags tmux` and `-tags llm`: not licensed by the board entry to extend, and a ban on a dead spelling is harmless.
- lens:code-affected-set, `internal/testkit/lyxbin/lyxbin.go:1` names the old spellings in a package doc: outside the diff and covered by the decision record's comment exemption.
- lens:code-affected-set, `docs/benchmarks/*` hits: measurement reports, stale by design.
- lens:code-tests finding 1, no multi-tier FileTier row: a multi-tier constraint is refused by the one-tag scan, and the removed row pinned only the retired spelling.
- lens:code-tests finding 2, three overlapping scan rows: each takes a different constraint shape, so none duplicates another.
- lens:code-tests finding 3, `fixtureIntegrationTests` and `integration_test.go` names: not a tag, and renaming is beyond the entry.
- lens:code-tests finding 4, `tags: "integration"` in `cmd/testtiming/splice_test.go` and `prebuild_integration_test.go`: the tags value is an opaque string to the code under test, so no tier is selected by it.
- lens:code-tests, no negative row that `tmux` is no longer the tmux tier in `fileTier`: optional negative, covered by the positive `tier2` rows.
