```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, for the teardown-resume skip (D1–D4).

**Per change, the set found:**

- **Failed-deletion error (`wire.go:384`):** the old wording was asserted in `wire_test.go:840` and `lifecycle_integration_test.go:705`. The diff updates both. The error text itself has no other asserters. The other "resume this run" strings (`wire.go:407`, `410`, `416`, `943`) belong to other rows, so they are out of scope. **Covered.**
- **`refuseResumeUnreachable` renamed to `refuseOwnedElsewhereOnLocalBoard`:** a grep finds no remaining reference to the old name outside `docs/research`, which is a historical record. The caller in `resumereconcile.go:92` and the test in `owner_test.go:260` are updated. **Covered.**
- **`ownerPreflight` skip:** its two callers are `preflightRunOwner` (`cli.go:452`, `arm.go:735`) and the batten bootstrap check (`arm.go:922`). Both reach the skip through the shared function, so the skip applies uniformly. **Covered.**
- **Tests that used Teardown as an arbitrary resume row:** `run_test.go` and `step_test.go` moved to `NameLand` so they still reach the reconcile. No other test in `internal/battencli` or `internal/dobbyengine` uses `NameWorktreeTeardown` with a status file in a way the grep exposed, though I only searched for the message strings. **Covered.**
- **Docs:** `docs/overview.md:704`, `cli.go` (package doc) and `arm.go:713` are updated. **Not covered:** the two items below.

**Findings:**

1. **MEDIUM, consistency.** `internal/battenshed/doc.go:59-64` describes the resume pre-flight as always running the run-scope origin reconcile first, and the local-board fallback as only for an unreachable origin. The diff adds an exception that this doc does not mention. The doc describes the same pre-flight as `cli.go` and `arm.go`, and the diff updates those two but not this one. Fix: add a sentence saying a resume at Worktree-Teardown whose task worktree is gone runs no reconcile and judges the owner on the local board alone.

2. **MEDIUM, consistency.** `pattern/PATTERN-owner-machine.md:20,27` says every hand pass is a resume and gets the origin reconcile, and that acting on a run starts with the reconcile. The skip is a new exception to that rule, and the rule document does not say so. Evidence: the diff touches neither `PATTERN.md` nor `pattern/`. The decision record's Constraints cite `PATTERN-owner-machine` as "judged on the local board when the reconcile is skipped", so the rule is expected to hold the exception. No declared PATTERN change exists in the rubric's input. This is a rule-document drift finding, not a missing declared entry. Fix: add a bullet to `pattern/PATTERN-owner-machine.md` noting the Worktree-Teardown, task-worktree-absent exception (local-board owner judgement, no origin-owed marker).

3. **LOW, consistency.** `owner_test.go:3` header says the file covers "the start check, and the owner keys of the status envelope". It now also covers the teardown pre-flight and the refusal exit codes, and `TestPassExitCode` was already there. Fix: extend the header comment to name the teardown pre-flight.

**Commit message and departures:** no `Pattern-Departure:` trailer is present and none is needed. The commit bodies are consistent with the diff.

**Tool calls:**
1. grep for the old and new message strings across the repo, to find asserters and docs I had not read.
2. Read `arm.go:900-940`, to confirm the second `ownerPreflight` caller.
3. grep of `PATTERN.md`, `pattern/`, `docs` for reconcile and resume statements, to find rule text describing the changed behavior.
4. Read `battenshed/doc.go:56-66` and `PATTERN-owner-machine.md:18-30`, to quote the resume descriptions finding 1 and 2 rest on.
