# Card 5 — plan gate PATTERN base wiring

**Create:**
- `internal/patternlint#OverviewAt`
- `internal/loomshed#PatternBaseReader`
- `internal/loomshed#TestPatternBaseReader`
- `internal/loomcli#TestGateParity_PatternHeadroom`

**Edit:**
- `internal/shedrecipe#planContextOf`
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/patternlint/doc.go`

**Uses:**
- `internal/patternlint#readOverview`
- `internal/loomcli#planFixture`
- `internal/loomcli#producerVerdict`
- `internal/loomcli#cliVerdict`

**Intent:**
D1: every plan gate and the `lyx loom validate-plan` self-check get the same merge-base reader, so the plan gate and the run-time headroom lint read one base.

`patternlint.OverviewAt(worktree, revision)` returns the overview's bytes at `revision`, nil when the overview is absent there, by delegating to `readOverview`; an empty revision is refused with an error, since the working tree is not a base; a merge base answering the empty string reaches it through `PatternBaseReader`.
`loomshed.PatternBaseReader(worktreeRoot, mergeBase)` returns a reader that, on each call, takes the revision from `mergeBase` when it is non-nil, and `HEAD` when it is nil, the standalone run with no parent recorded, and reads `OverviewAt` at it; a `mergeBase` error is returned wrapped.
It reads on demand and opens nothing when built.

`shedrecipe.planContextOf` sets `PatternBase: loomshed.PatternBaseReader(env.WorktreeRoot, env.VerifyMergeBase)`, the same merge-base reader the verify gate's headroom lint is told, so the Plan-Write, Plan-Burler-Fix and PR-Rework gates read the base the verify gate does.
`validatePlanCmd` sets the same field from `c.env.WorktreeRoot` and `c.env.VerifyMergeBase` in every mode that builds a `PlanContext`, so the verb and the row gates give one verdict.
`internal/patternlint/doc.go` names `OverviewAt` as the shared base read.

Both tests build git repositories, so each sits in an `//go:build integration` file of its package (tier 1), as the package's other git-backed tests do.
`TestPatternBaseReader`, in package `loomshed`, builds a hermetic git repository (`gitkit.HermeticGitEnv()`, PATTERN-test-isolation) with `PATTERN.md` committed at a first commit and grown at a second, and asserts: a nil merge base reads `HEAD`'s bytes; a merge base naming the first commit reads its bytes; a revision whose tree has no `PATTERN.md` reads nil; a merge-base error is a returned error; a merge base answering the empty string is the returned error of `OverviewAt`'s empty-revision refusal.
Card 4's tests drive the checks through a fixed reader, so the wiring alone is pinned here.
`TestGateParity_PatternHeadroom`, in package `loomcli`, pins the criterion that `lyx loom validate-plan` and the row gate give one headroom verdict on a branch that already grew `PATTERN.md`: on a `planFixture` whose worktree is a hermetic git repository with `PATTERN.md` committed under the headroom line at a first commit and grown at a second, still under the line, with `c.env.VerifyMergeBase` answering the first commit and a card fencing a line that grows the projection past the line, it builds the gate with `loomshed.PlanContext{PatternBase: loomshed.PatternBaseReader(c.env.WorktreeRoot, c.env.VerifyMergeBase)}` and runs `validate-plan`, and asserts both `producerVerdict` and `cliVerdict` are stuck naming `pattern-headroom`; a second case whose fenced line replaces a bullet at equal size, keeping the projection under the line, passes on both.

**ImpactSummary:** `planContextOf` serves the three plan gates and `validatePlanCmd` the verb; both only gain the field, and with no `PATTERN.md` in a worktree the checks read nothing, so existing gate and parity fixtures see no change.
