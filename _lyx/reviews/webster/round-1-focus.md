---
round: 1
focus:
  - "Ownership boundary: in the committed diff, look for any edit under `internal/fabricengine`, `internal/websterengine`, `internal/reedengine` (outside `render`), `internal/shuttleengine` (outside `claudeengine`), `internal/loomcli`, `internal/shedadapters`, `internal/testkit/`, `cmd/testtiming` or an `llm`-tagged file, and settle whether production code changed beyond deleting a caller-less non-test helper."
  - "Protected tests: settle whether any test named by a core or testkit file, or any skill-loading test in `internal/shuttleengine/claudeengine` (`Spec.Skills`, `SkillLoadSequence`, `SkillUnknown`, `DefaultSkillLoadTimeout`, `loadSkill`, `loadSkillsThenPrompt`, the `SkillLoader` fakes), was deleted, renamed, folded, merged or made a scenario step, and whether a kept skill-loading candidate carries `//testtiming:keep skills-in-one-turn`."
  - "Keep directives: settle whether every `//testtiming:keep` added sits directly above its `func Test…` line with a non-empty reason that is one of the plan's reason classes, and whether any candidate was annotated without a qualifying reason instead of being deleted or folded."
  - "Assertion preservation: for each deleted candidate, fold, scenario merge and tier move, settle whether every assertion it replaced survives in a covering or receiving test, comparing the removed assertions against the named receiver in the commit message."
  - "Scenario merges: settle whether each merged scenario runs its `t.Run` steps serially with `if !t.Run(...) { return }`, comments the steps that depend on earlier state, and calls `t.Parallel` at top level with a comment naming the shared fixture; and whether it removes fixture builds as the commit message claims."
  - "Tier moves: settle whether a test moved from an `integration` file to an untagged one truly needs no git, tmux or subprocess, uses an existing fake, and otherwise carries `// tiermove: needs shared fake <name>`; and whether `gitkit.HermeticGitEnv()` and `tmuxkit.Main` remain where `PATTERN-test-isolation` requires them."
  - "Folds into tables: settle whether each table-driven test is named after the behavior and uses `t.Parallel` at top level and per subtest, or a comment naming the global state that forbids it."
  - "Leftovers: settle whether a helper with no caller under any tag (`llm` included) or an emptied test file remains, whether a `TestMain` was wrongly removed, and whether docs, `pattern/` files, stencils, specs or comments naming a deleted or renamed test were updated in the same commit."
  - "Per-card mechanical checks and verify: settle whether each card's own commit ran its `go run ./cmd/testtiming -redundancy` refresh with `-out` under `.scratch/`, and whether the final tree passes the plan's `verify:` commands across the default, `integration` and `tmux` tags."
  - "Card 1 and card 23 measurement: settle whether the before report was recorded on the untouched tree, and whether the rerun spliced only in-scope sections into `docs/benchmarks/test-redundancy.md` and recorded the after measurement consistently with the before one."
  - "Comment conventions: settle whether new or changed comments follow `PATTERN-comment-line-breaks` and the repository's own constraints document, with no line width inferred from surrounding code."
---

Focus follows the plan's card index and its `prune-procedure` and `ownership-boundary` decisions.
Entries name where to look; none caps severity or pre-states a verdict.
