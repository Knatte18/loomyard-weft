---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/websterengine/git.go:2
    summary: New Go doc comments and scenario step comments keep two clauses or two sentences on one line, against PATTERN-comment-line-breaks
  - id: F2
    severity: LOW
    class: consistency
    location: docs/benchmarks/test-suite-prune-core.md:12
    summary: New markdown joins independent clauses on one line instead of breaking at the semicolon or the comma before a conjunction, against CLAUDE.md semantic line breaks
---

Range: `product.parent` is `main`, no `_lyx/loom/rework/` round exists, so the range is `git merge-base main HEAD..HEAD`, 17 commits, one per card.
`go build`, `go vet` under every tag, `go test ./...`, and the plan's `integration` and `tmux` test commands over the six packages, `cmd/` and `internal/testkit/...` all pass at `HEAD`.

Settled against the focus directive, none of which is a finding:

- **Seam changes.**
  Every production hunk in `internal/websterengine` replaces a direct helper call with the same helper reached through `Geometry.git()`, which returns `realGit{}` when `Geometry.Git` is nil.
  The new surface is one struct field, `Geometry.Git`, and one field on `batchFailInputs`; no package-level variable and no default other than today's behavior was introduced.
  No production file outside `git.go` and `gitwrap.go` still calls a wrapped helper directly.
  `mergeRejection` resolves the parent tips on each merge it checks where the old walk resolved them once; the result and the error text are the same, so it is not a behavior change.
  `internal/fabricengine/mergeerrors.go` changes a test name in a comment only, and `envkit.go` gains the one `nilLegal` row the new field needs; card 9's commit message names that kit path as the untag decision requires.
- **Write scope.**
  Every card's commit stays inside its Targets, plus `internal/testkit/envkit/envkit.go` in card 9, which the `untag-procedure` kit allowance covers.
- **`skill-loading-tests-untouched`.**
  The bodies of `TestPrepare_SkillsDeferPromptPointer`, `TestRunner_LoadSkillAndSkillUnknown_*`, `TestSkillLoadSequence`, `TestSkillUnknown`, `TestStartup_SkillSkipCauses`, `TestStartup_SkillsLoadInOrderBeforePrompt`, `TestStartup_SkillsRefusedWithoutSkillLoader`, `writeStubDriverScript` and `integrationWriteStubDriverScript` are byte-identical to the merge base, and `fakes_test.go`, `internal/testkit/shuttlefake` and `internal/shuttleengine/claudeengine` are untouched.
  The two kept candidates, `TestStartup_SkillsLoadInOrderBeforePrompt` and `TestStartup_NoSkillsNoPromptLineSendsNothing`, carry reasons naming `skills-in-one-turn`.
  The `Spec.Skills` assertions in `internal/shedadapters` and `internal/websterengine` were folded into tables, which the decision's list (`internal/shuttleengine` and `internal/shuttleengine/claudeengine`) does not cover; each assertion is kept.
- **Test economy.**
  No deleted test installs a git hook, alias or helper, so D4's hand check has nothing to catch.
  The sampled folds (`TestMutations_NilReceiver_DoesNotPanic`, the `TestOpen_Missing*` pair) keep their assertions in a named covering test.
  Each regenerated section lists no candidate outside "Kept" and no failed test, and the "Kept" reasons name what the test pins.
- **Folds, scenarios, tiers.**
  The scenario tests keep `t.Parallel`; no untagged file spawns a process, starts tmux, uses real git or hand-builds a hub; every package that keeps tagged files keeps `tmuxkit.Main`, and the git-spawning ones keep `gitkit.HermeticGitEnv()`.
  Folded tests in `shedadapters`, `shuttleengine`, `reedengine` and `loomcli` have no `t.Parallel` because the tests they replace had none; that is not new in this diff.
- **`cmd/testtiming`.**
  The exclusion rules match card 2, the keep scan runs before any measurement, the splice leaves the header and unmeasured sections untouched and refuses a different tag set, and each rule has a test row.
- **Report.**
  The after table has the before table's shape plus a delta table, and the lost-coverage section names no lost block.
  Timing run 1 recorded one undiagnosed `internal/reedengine` failure under contention that did not reproduce; the report says so, and nothing in the diff points at its cause.

### [LOW] Doc and step comments break the line-break rule

**Location:** `internal/websterengine/git.go:2`, `internal/websterengine/geometry.go` (the `Git` field comment), `internal/websterengine/batchfail.go` (the `Git` field comment), `cmd/testtiming/main.go`, `cmd/testtiming/redundancy.go`, `cmd/testtiming/spawnscan.go`, the scenario header and step comments of `internal/fabricengine/*_integration_test.go`, `internal/loomcli/smoke_bootstrapwiring_test.go`, `internal/loomcli/wiring_test.go`.
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line and a break at an internal independent clause, a semicolon or a comma before a conjunction with its own subject and verb.
Comments written or rewritten by this task keep `; a nil one means realGit` clauses on the sentence's line, put two sentences on one line (`The first `step` is a genuine first seed. It dispatches Preflight…`), or hard-wrap the scenario step comments at a column.
**Fix:** Reflow each comment this task wrote or changed to one sentence per line, breaking at the semicolon or conjunction clause; leave comment text carried over unchanged from the merge base alone.

### [LOW] Markdown joins independent clauses on one line

**Location:** `docs/benchmarks/test-suite-prune-core.md`, `docs/benchmarks/running-tests.md` (the lines added by card 2 to 4).
**Issue:** `CLAUDE.md` requires semantic line breaks in every `.md` file.
The measurement report puts `;` and `, and` clauses with their own subjects on one line (`Wall is the package's elapsed time…; serial is the sum…`, `the three timing runs started at rising load, and run 1 was the quietest`, `its test name was not captured, and the package passed…`), and the added `running-tests.md` paragraph does the same.
**Fix:** Break those lines at the semicolon or the comma before the conjunction; change no wording.
