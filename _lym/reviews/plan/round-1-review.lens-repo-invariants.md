```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. I checked each card for reflective-seam coverage, new test-file tier tags, doc text the change falsifies, and card order.

**Per card**
- **Card 1:** tier tag held (`queue_test.go` is tier0). Seams held, since it adds no `Deps` field. `internal/lymdirs/doc.go` lists each hub-level name, and the new `LandingQueueDirName` is missing from that list: broken, LOW (F5).
- **Card 2:** the gitrepo CLI boundary pin list is broken: BLOCKING (F1).
- **Card 3:** tier tag held (`landingchain_integration_test.go` is tier1). Docs held.
- **Card 4:**
  - `internal/fabricengine/doc.go:141` becomes false: broken, MEDIUM (F3).
  - The new test `TestPushPairAnchored_HoldsOffTheCodeSideOfADetachedPair` names no tier and no file: broken, LOW (F6).
  - The checkout verbs name a gitrepo route that does not exist: broken, MEDIUM (F2).
  - `lymdirs/doc.go` misses `DetachMarkerName`: broken, LOW (F5).
- **Card 5:** card order held, since `Config.FinalizeLostRaces` stays in place for card 6 and the remaining tests still compile. Tier held (`finalize_unlocked_test.go` is tier0). The `landingshed/doc.go` text is left for card 12: held, since docs carry no gate.
- **Card 6:** card order held, since every `FinalizeLostRaces` reader is in its targets or card 5's. Seams held.
- **Card 7:** the seam markers on envkit's `NilSeams` tests are broken: BLOCKING (F4). The other seam-table entries hold. Tier held (`finalize_queue_integration_test.go` is tier1).
- **Card 8:** the seam markers for `DetachDeps` on the same envkit tests are broken: BLOCKING (F4). `TestLandingDeps_TaskHeadReadsTheBranchWhileDetached` is tier1 in a package whose `landingdeps_test.go` is tier0, and the card names no file: LOW (F6). Card order held.
- **Card 9:** card order held, since it uses card 8's `gateChainCommit` and `landPrefix`. Tier held.
- **Card 10:**
  - Seams held: no reflective test marks `internal/shedengine#Shed`, and `DetachState` is a func field covered by the listed tables.
  - `internal/shedengine/doc.go:27-28` names `LockedCheck` as the only run-lock hook and the card adds a second: LOW (F7).
  - `TestNewShed_RestoresADetachedTaskWorktree` is tier1 in loomcli with no file named: LOW (F6).
- **Card 11:** held, since it uses card 1's `Store.Backstop` and `fabriccli/doc.go` is under `Extends`.
- **Card 12:** held. It is the last card and covers `PATTERN.md`, `pattern/PATTERN-landing.md`, `landingshed/doc.go` and the glossary. No other `pattern/` file or `PATTERN.md` line mentions the lock-free pass or lost races.

**Findings**

- **F1 — BLOCKING, consistency, card 2 (`plan:internal/gitrepo#Repo.WriteCommitTree`).** `WriteCommitTree` calls `r.runChecked`. That puts it on `gitrepoPinnedRunBoundMethods`, which `TestGitrepoBoundary_PinnedRunCallSites` (tier0) holds equal to the set of methods that call `run` or `runChecked`. No card edits that list.
  - The card's plan to read the message "from standard input" makes it worse. `runChecked` (`internal/gitrepo/gitrepo.go:83`) passes no stdin. Calling `gitexec.RunStdin` directly would break the test's second assertion: exactly two `gitexec` call expressions, inside `run` and `runChecked`.
  - Evidence: `cmd/lym/gitrepoboundary_test.go:22-25,56,69-71`; `internal/gitrepo/gitrepo.go:83`.
  - Fix: list `cmd/lym/gitrepoboundary_test.go`'s `gitrepoPinnedRunBoundMethods` under card 2's Edit. Pass the message with `-m` through `runChecked`, or route stdin through a choke point.
- **F2 — MEDIUM, consistency, card 4 (`Fabric.CheckoutDetached`, `Fabric.CheckoutBranch`).** Both say they run `git checkout` "through `gitrepo`". No gitrepo method runs `checkout` or `--detach`, and no card creates one. Only `MergeFFOnly` and the `Merge*` methods exist, so `MoveCodeTo`'s `--ff-only` move is fine.
  - A new gitrepo method would also have to join the pin list from F1.
  - Evidence: grep of `internal/gitrepo/*.go` for `Checkout*`, `--detach` and `--ff-only`; only `merge.go:266,270` and `pull.go:33` match.
- **F3 — MEDIUM, consistency, card 4 (`internal/fabricengine/doc.go`).** The package doc says "`RunCommitsAhead`, `RefusedPushStands` and `TipsOnOrigin` read local refs and failed-push records only". Card 4 makes them read the detach-marker file under the worktree's `.lym`, so the sentence becomes false. The doc is only in Uses.
  - Evidence: `internal/fabricengine/doc.go:141`.
- **F4 — BLOCKING, consistency, cards 7 and 8 (`internal/testkit/envkit#TestNilSeams_ReportsExactPaths`, `#TestNilSeams_SkipsNilLegalPaths`).** Both tests reach `envkit.NilSeams` and carry `//lym:seam internal/landingshed#Deps`. The tier0 scan refuses "a test reaching `envkit.NilSeams` whose markers miss a struct the `Env` set holds". Card 7's `Deps.Backstop` (`landingqueue.BackstopSeams`) and card 8's `Deps.Detach` (`landingshed.DetachDeps`) nest new structs.
  - Both cards list these tests under Uses and leave them out of the marker list in their Intent. `seam-test-unlisted` passes a test listed under Uses that the card breaks, so the gate does not catch this.
  - Evidence: `internal/testkit/envkit/envkit_test.go:115,167`; `pattern/PATTERN-test-speed.md:174-181`; `loom-plan-checks-spec.md:271`.
  - Fix: move both tests to Edit in card 7 for `//lym:seam internal/landingqueue#BackstopSeams`, and in card 8 for `internal/landingshed#DetachDeps`.
- **F5 — LOW, consistency, cards 1 and 4 (`internal/lymdirs/doc.go`).** The package doc names each lymdirs constant: the hub-level names, then the names under a worktree's `.lym`. `LandingQueueDirName` and `DetachMarkerName` join neither list, and the doc is only in Uses.
  - Evidence: `internal/lymdirs/doc.go:6-15`.
- **F6 — LOW, design, cards 4, 8 and 10 (new test functions with no file).** Each test file's functions must name one tier.
  - Card 4's `TestPushPairAnchored_HoldsOffTheCodeSideOfADetachedPair` states no tier and no file.
  - Card 8's `TestLandingDeps_TaskHeadReadsTheBranchWhileDetached` and card 10's `TestNewShed_RestoresADetachedTaskWorktree` are tier1 in loomcli, whose `landingdeps_test.go` is tier0, and neither card names a file.
  - Evidence: `pattern/PATTERN-test-speed.md:24`; tier0 at `internal/loomcli/landingdeps_test.go:49`.
  - Fix: name a tier1 file for each, such as an `_integration_test.go`.
- **F7 — LOW, consistency, card 10 (`internal/shedengine/doc.go`).** The package doc describes `Shed.LockedCheck` as the hook that runs under the run lock before the status file is read. Card 10 adds `Shed.Restore` at the same point, and it changes the worktree. The doc is only in Uses, so the documented drive order goes incomplete.
  - Evidence: `internal/shedengine/doc.go:27-28`.

**Tool calls**
1. `fabricengine/doc.go` reads mentions and `shedengine/doc.go` lines 20-40: to check the doc text cards 4 and 10 change, which the manifest did not hold.
2. A repository search for lock-free and lost-race wording, plus the tier-file rule in `PATTERN-test-speed.md`: for docs outside card 12's targets and the one-tier-per-file rule.
3. `lymdirs/doc.go` names and the other lock-free hits: to see whether the doc enumerates the names cards 1 and 4 add.
4. `PATTERN-gitrepo-client-boundary.md` pinned-site lines and the location of the pin-list test: card 2's new CLI method.
5. `cmd/lym/gitrepoboundary_test.go` header and pin list: to confirm the test asserts an exact method set.
6. gitrepo checkout and fast-forward methods: whether card 3's and card 4's "through gitrepo" verbs exist.
