---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/orchengine/watcher_test.go:908
    summary: TestWatcher_SkillSkipCauses selects its final assertion by a bare string, "reloaded" or "skippedOnly", while its sibling table in startup_test.go uses named constants; a typo silently skips the assertion
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shuttleengine/startup_test.go:973
    summary: TestStartup_SkillSkipCauses switches on tt.elapsed with no default, so an unknown value runs no elapsed assertion
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shuttleengine/session_test.go:183
    summary: the rewritten guard test and the two rewritten skip-cause tables neither call t.Parallel nor name the process-global state that forbids it, as PATTERN-test-parallel requires
---

The diff covers the five commits after the rejected head `fbcf5b3`.
`go build`, `go vet -tags 'integration smoke'` and the tests of every touched package pass.

Judged against the plan and the focus directive:

- Card 17: every statement in the orch role stencil matches the recipes and docs I read.
  `batten-recipe.yaml` runs Worktree-Create, Seed-Child, Run-Shed, Worktree-Teardown.
  `internal/battenshed/doc.go` and `deps.go` confirm that batten never resumes a halted child, resumes an awaiting child on an approve or reject decision (`ReadDecision`), and tears the pair down at the end.
  `internal/landingshed/doc.go` confirms Finalize's sync, squash, board-done, push and PR close.
  The `wordingClaims` row pins each corrected fact and each removed phrase.
- Cards 18 and 19: each removed per-phrase test has a claim row, and the loom-row scan over the Go-declared `loomshed.InterruptPolicies` stays a scan in `cmd/lyx/driverstencil_test.go`.
  The heading, no-scribe-handoff and note-template checks are `wantAll`/`wantNone` rows; the leading-comment check stays in the render table.
- Card 20: `Runner.ContextTokens` and `CompactedSince` keep their plain-engine refusal test, and the engine side keeps its own tests in `claudeengine/usage_test.go`.
  Nothing references the deleted tests or the deleted getter test; `DefaultSkillLoadTimeout` is still reached through `run.go` and the startup tests.
- Card 21: both tables keep the two causes as separate cases, each with its own expected values.
- Per-card mechanical check: the `Delete` groups of card 20 leave no caller of the removed tests (grep over the tree).
- Test economy: no new test duplicates an existing one.

### [LOW] Stringly-typed final-assertion selector in the orch table

**Location:** internal/orchengine/watcher_test.go:908
**Issue:** `final` is a bare string checked against `"reloaded"`; any other value falls to the `else if` branch, so a typo changes which assertion runs without failing.
The shuttle table declares constants for the same device.
**Fix:** declare named constants and use them in the cases and the comparison, as `startup_test.go` does.

### [LOW] Elapsed switch without a default

**Location:** internal/shuttleengine/startup_test.go:973
**Issue:** the `switch tt.elapsed` has no `default`, so a mistyped case value runs no elapsed assertion and the test still passes.
**Fix:** add a `default` that fails the test on an unknown value.

### [LOW] No t.Parallel and no named global state

**Location:** internal/shuttleengine/session_test.go:183
**Issue:** `PATTERN-test-parallel` requires `t.Parallel` unless the test touches process-global state, with a comment naming that state.
The rewritten `TestRunner_LoadSkillAndSkillUnknown_GuardStrands`, `TestWatcher_SkillSkipCauses` and `TestStartup_SkillSkipCauses` do neither; they use a fake clock and fakes over temp directories.
**Fix:** call `t.Parallel` in each rewritten test, and in its subtests where the table has them, after confirming the fixtures share no global state.
