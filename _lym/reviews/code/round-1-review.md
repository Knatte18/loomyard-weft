---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/prfindings_integration_test.go:37
    summary: The regression test calls the unexported prReviewSendBackEdge directly instead of driving the pr-findings verb, against PATTERN-test-speed's rule that an unexported helper is tested through the surface that uses it, so reverting the command's edge to an unwired closure passes every test
    fix: In internal/loomcli/prfindings_integration_test.go, replace the direct c.wire plus c.prReviewSendBackEdge calls with a run of the verb through clihelp.ExecuteIn(Command(), <the pair's code worktree>, &out, []string{"pr-findings", <findings file>}), the way circling_integration_test.go drives its verb. Before the run, write a pull-request record at shedrun.PullRequestRecord(location, shedrun.SelfRunID) through the pullrequest package, and write a findings file in burler's review format under the worktree's .scratch/ directory, so the verb reaches its edge call (prfindings.go:199). Assert the ok envelope the verb prints, not an error envelope naming the edge. Keep the tier-1 tiergate.Require, the stencil seeding and the tapestry recipe. Rename the test to name the verb's behaviour, and delete the helper-level test rather than keep both.
    origin: assist:assist
  - id: F2
    severity: LOW
    class: consistency
    location: internal/loomcli/sharedbootstrap.go:437
    summary: The comment above the sampler branch of wireLanding says only the building paths pass through it, so the record watch opens only for a run that drives the row; the pr-findings edge now passes through it on a sampler run, whose recipe holds both a PRReview and a SamplerSession row, and opens the watch without driving the row
    fix: In internal/loomcli/sharedbootstrap.go, replace the two comment lines "The building paths pass through here; status and pause never do." and "So the watch opens only for a run that drives the row." with lines saying that the building paths and pr-findings' send-back edge pass through here, that status and pause never do, and that the edge's process exits right after it reads the shed, so its watch is never waited on.
    origin: lens:code-affected-set
  - id: F3
    severity: LOW
    class: consistency
    location: internal/loomcli/prfindings_integration_test.go:24
    summary: The test builds an empty hub and adds a fixed-slug pair with AddPair on every run, though the closed shape set already holds a pair-carrying hub
    fix: "In internal/loomcli/prfindings_integration_test.go, copy hubforge.Shape{Anchor: \".\", Pairs: []string{hubforge.TemplatePairSlug}} with hubforge.CopyHub, drop the slug constant and the hubforge.AddPair call, and resolve the worktree from hub.PairCodeWorktree(hubforge.TemplatePairSlug)."
    origin: assist:assist
---

### [BLOCKING] The regression test bypasses the verb it protects

**Location:** internal/loomcli/prfindings_integration_test.go:37

**Issue:** `PATTERN-test-speed`'s background (`pattern/PATTERN-test-speed.md:244-246`) says a test targets the public surface, a CLI verb included, and that an unexported helper is tested through the surface that uses it.
The new test calls `prReviewSendBackEdge` and `wire` directly.
`TestPRFindingsVerb`'s rows inject a fake edge (`prfindings_test.go:384`).
The defect sat in the seam between `prFindingsCmd` and the edge.
If the command went back to an unwired closure while the method stayed correct, every test would still pass.
The code-tests lens raised the same gap as a NIT on `prfindings.go:343`, and it is merged here.
The verb calls its edge for every findings set once a pull-request record exists (`prfindings.go:187-199`), so a tier-1 run through `clihelp.ExecuteIn` reaches it.
A violation of a listed constraint is BLOCKING under this round's rubric.

### [LOW] The sampler-watch comment is stale since pr-findings passes through wireLanding

**Location:** internal/loomcli/sharedbootstrap.go:437

**Issue:** `contracts/recipes/sampler-recipe.yaml` holds both a `SamplerSession` row (line 105) and a `PRReview` row (line 244).
On a sampler run, `prReviewSendBackEdge` passes the row check and calls `wireLanding`, which runs `openRecordWatch` (`samplerwiring.go:91-98`).
The comment at `sharedbootstrap.go:437-439` says the watch opens only for a run that drives the row, which is no longer true.
Filling the sampler's session seams is needed, since the SamplerSession constructor validates every `Env.Sampler` seam but `Watch` (`internal/shedrecipe/entries_samplersession.go:19`).
The watch itself lives only for the verb's short process, so the code needs no change.
The code-correctness lens reported the same site as a LOW `design` finding at `prfindings.go:298`, and it is merged here.

### [LOW] The test builds its pair per run instead of copying the pair-carrying shape

**Location:** internal/loomcli/prfindings_integration_test.go:24

**Issue:** `pattern/PATTERN-test-isolation.md:212-216` names pairs under fixed slugs as a start state for a template.
`hubforge.Shapes` already carries a hub with the one pair `TemplatePairSlug` (`internal/hubforge/template.go:33-56`).
The test copies an empty hub and runs `AddPair` under its own slug on every run, which the copied shape makes unneeded.
The background's rule binds start states the closed shape set cannot hold, and this one it can hold.
So this is a cost and consistency finding, not a constraint violation.

## Rejected

- code-tests LOW at `prfindings.go:294`, no test for the no-PR-Review-row answer: the branch and its text predate the diff, which moved it unchanged out of the closure, so it is no behaviour the diff adds.
- code-tests NIT at `prfindings.go:298-300`, no test for a `wireLanding` error: the branch returns the error unchanged, the same shape as the `newShed` error path beside it, and the verb's read-failure envelope for any edge error is already covered through the fake edge.
- code-tests NIT at `prfindings.go:343`, the command's edge seam is not pinned: merged into F1.
- code-correctness LOW at `prfindings.go:298`, the edge opens the sampler watch: merged into F2. The watch is a process-lifetime watcher in a verb that exits after it answers, and the sampler seams it fills beside the watch are required for the shed to build, so only the stale comment remains a defect.
- The assistant rated F3 BLOCKING as a `PATTERN-test-isolation` violation: the background's registered-template rule binds start states the closed shape set cannot hold, and `hubforge` documents `AddPair` for verb tests that need a pair, so it is kept at LOW.

## Focus questions settled

The findings and rejections above settle the focus questions, together with these notes.
`wireLanding` fills every seam the landing rows' constructors check, including `OpenFabric` and `OpenParentFabric` (`landingdeps.go:115-120`), and a second call overwrites `Env.Landing` whole.
No other caller of `newShed` builds a shed without first wiring the landing seams.
`status` and `pause` leave `BuildShed` nil (`arm.go:465-474`).
The new test fails on the pre-fix code, because the tapestry recipe holds a Finalize row whose constructor refuses a nil `OpenFabric`.
Both edited doc lines in `doc.go` match the code.
