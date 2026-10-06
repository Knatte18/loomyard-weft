# Card 9 — websterengine-untag

**Edit:**
- `internal/websterengine#`

**Intent:**
Run the untag procedure (Shared Decisions, `untag-procedure`) on `internal/websterengine`'s `integration` files.

Read each `integration` test in turn and ask what it needs real git for.
Webster's batch lifecycle (`beginbatch`, `recordbatch`, `recoverbatch`, `runlevel`, `suspect`, `rebaseline`) reads and writes git through `internal/fabricengine` and `gitrepo`; a test whose assertions are about webster's own records, prompts, warnings or verdicts and not about the git state may move to an untagged file over a fake of the git-facing dependency, where the engine already takes that dependency as a field or option.
Where it does not, add the seam as a struct field or option that production sets to today's value (D6), in the same card.
A test whose behavior is the git interaction stays `integration`.

Then merge scenario families (Shared Decisions, `scenario-merge`) among the tests that stay tagged: `runlevel_test.go`, `recordbatch_test.go` and `recoverbatch_test.go` hold most of the package's integration tests, and many of them build one scratch git repo with `gitkit.CommitFile` (for example `newRecordFixture`, `newScratchRepo`) plus its plan, report and webster temp directories, and differ only in the steps after it.
That fixture is a seeded worktree, not a hub: no webster test imports `internal/hubforge`, and `PATTERN-hubforge-fixtures` does not apply to it, so a merge keeps the scratch-repo fixture rather than rebuilding it on `hubforge`.
The commit message names each merged family and how many fixture builds it removed.

The package keeps `tmuxkit.Main` and `gitkit.HermeticGitEnv()` in `TestMain` while any tagged file remains.
Regenerating the report section is the prune card's job, not this card's.

**ImpactSummary:** Test files of one package, plus a shared fake under `internal/testkit/` where `untag-procedure` allows one, plus a seam field in its production code only where a moved test needs one; production passes today's value, so no caller changes behavior.

**Verify:**
go vet -tags tmux ./internal/websterengine/
go vet -tags llm ./internal/websterengine/
