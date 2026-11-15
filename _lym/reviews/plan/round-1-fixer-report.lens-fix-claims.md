```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: I checked whether each sentence the fix added or rewrote holds against the code at HEAD. One claim fails.

**MEDIUM: card 9's list of `Teardown.Run` callers is incomplete**
- **Changed text:** the new Intent bullet in card 9 ("Every `Teardown.Run` caller reaches the step: `lym fabric remove` and the origin-reconcile teardown (`teardownEnded`)…") and the rewritten ImpactSummary ("reached by every `Teardown.Run` caller: `lym fabric remove` and the origin-reconcile teardown").
- **Summary:** these lines say two callers reach the new post-removal step, but a third production caller calls `Teardown.Run` directly.
- **Evidence:** `internal/battencli/beamfork.go:176-180` builds `pairteardown.New(prime)` and then calls `td.Run(ctx, pairteardown.Request{Slug: slug, Remote: true, LeaveClaim: true})`. That is the beam fork's member removal, a third path to the cancelled-record deletion that both sentences leave out.

Claims that hold:
- **Card 12:** loom run and step ask neither the pause nor the owner refusal today. `refuseCovered` and `refuseOwnedElsewhere` are called only from `start.go:531,535` and `resume.go:272,276`.
- **Card 4:** `TaskBranchOnOrigin` runs `git ls-remote`. The path is `originTaskBranch` → `firstOnOrigin` → `remoteHeadTip` (`pairbranch.go:137-138`, `remoteleftover.go:236`).
- **Card 5:** a test-only import of `battenrecipe` closes no cycle. `go list -deps ./internal/battenrecipe` reaches `preflightshed` but not `pairteardown`, and only `preflightshed`'s test files import `hubforge`.
- **Card 6:** `TestBattenIntegration_Rows` is tier1 (`tiergate.Require(t, 1)`, `lifecycle_integration_test.go:315`). There are exactly three doc comments: `battenCLI`, `RunStatus` and `runStatusSeparator`.
- **Card 7:** `endRunPhase` asks the verdict after `judgeOrigin` and the reached-parent check, and before `refuseNothingLeft` (`cancel.go:298-330`). So a board `done` slug with no batten record does reach the verdict. `judgeOrigin` refuses an owned-elsewhere slug on every path (`cancel.go:553-572`).
- **Card 8:** today the marker is written while the lock is held (`cancelmarker.go:137-159`).
- **Card 11:** `strandedEnded` calls `drivable` (`ended.go:36`), so keeping the paused clause there is needed. The lift subtest is inside `TestDrivePasses_ExitHandling` (`passes_test.go:539`). The allowlist test fails on any import not on its list.

Tool calls:
1. Grepped `originTaskBranch`, ran `go list -deps` on `battenrecipe`, read the tier line of `TestBattenIntegration_Rows` and the order of the cancel refusals. My context had neither the import graph nor these code spans.
2. Read `endRunPhase` up to the reached-parent check, to confirm where the verdict sits relative to the nothing-left refusal.
3. Grepped the `pairteardown.New` and `Teardown.Run` callers, to check card 9's caller list.
4. Read the rest of `childReachedParent` and the `Run` call in `beamfork.go`, to confirm the third caller and the reached-parent rule.
