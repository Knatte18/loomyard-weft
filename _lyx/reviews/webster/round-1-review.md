---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/gitrepo/blobread_integration_test.go:233
    summary: no test pins the 24-hour slack of the HeadContains bound, so a zero slack or a wrong constant passes every test
  - id: F2
    severity: LOW
    class: consistency
    location: PATTERN.md:36
    summary: the PATTERN-hub-store-gc entry says only the pair-removal housekeeping writes the keys, but Topology.Add writes them too
---

The range is `git diff $(git merge-base main HEAD)..HEAD`: `product.parent` in `_lyx/shed/git-robustness/status.json` is `main`, and `_lyx/loom/rework/` does not exist, so no live rework round applies.
The range holds the seven commits for cards 1 to 6, plus the follow-up commit that declares `ConfigureGroupKill` once.

Verification run in this worktree, all clean:

- `go build ./...`
- `go vet`, `go vet -tags integration`, `go vet -tags tmux` and `go vet -tags llm`
- `go test ./...`
- `go test -tags integration ./...`
- `GOOS=windows go vet` over `internal/proc`, `internal/gitexec` and `internal/verifyrun`

Focus items settled:

- **go-git reads.**
  `goGit()` has one production caller, `readGoGit`, and `gogitcaller_test.go` pins that.
  Every listed method now runs its whole read in one closure, or in two sequential `readGoGit` calls for `HeadContains` and `CommitParents`.
  `readGoGit` snapshots `lastPackFingerprint` under the shared lock and reruns the whole `read` after a reindex.
  The fingerprint is compared against that snapshot, never against `lastPackFingerprint` alone, which closes `d1-split-lock-retry-race`.
  `lookupObjectRetrying` has no remaining reference.
- **`HeadContains` bound.**
  It walks with `LogOrderCommitterTime` and stops once a commit is more than `headContainsCommitterSlack` (24h) older than the target's committer time.
  The target hash is tested before the bound, so a target at the boundary is found.
  A walk not-found stays a wrapped error, as card 2 requires.
  The skew limit is stated in the doc comment as card 1 asks.
  F1 is the one gap.
- **proc and gitexec.**
  The verifyrun `procgroup_*.go` files are deleted and `configureProcessKill` has no remaining caller, so the kill was moved, not copied.
  `verifyrun` logs the same three messages through its `report` closure.
  `runCore` gives every `remoteSubcommands` member (now including `pull`) the deadline, `proc.ConfigureGroupKill`, `WaitDelay` and `lowSpeedEnv`.
  A timeout returns a `*GitError` with `Timeout` set, which `IsTransportFailure` reports as true only for a remote subcommand.
  Local commands run unchanged.
- **Hub store gc.**
  `disableStoreAutoGC` writes both keys in both stores.
  `Topology.Add` calls it after the pre-flight refusals and before the first mutation, and `Remove` runs `housekeepStores` only on the success path.
  Every gc failure is a logged warning that never fails the caller, and the integration test pins that with `gc.bigPackThreshold=bogus`.
  `housekeeping.go` is a separate file from `destroy.go`, and the destructive-guard test passes.
- **batten `--window`.**
  `runInWindow` calls `checkRunStatus`, the same function `battenPreRun` now calls, before `windowOpener`.
  `shedPaths` is populated by `arm` in the persistent pre-run, so it is set by then.
  A done slug never reaches the opener, and `TestRunInWindow` asserts that.
- **Per-card mechanical checks.**
  Card 2's `Delete` target `lookupObjectRetrying` and card 3's deleted files have no remaining callers.
  The `Create` targets exist, and `PATTERN.md` stays within `MaxOverviewBytes`.
  Every clause removed from an existing `PATTERN.md` entry is still stated in that entry's background file.
- **Test economy.**
  The budget rows moved by the net new top-level tests: `internal/gitrepo` +1, `internal/gitexec` +1 and `internal/fabricengine` +1.
  `battencli` adds rows only, and has no budget change.
  `TestSHAExists_MixedBackend_RepackBetweenCommitAndRead` is absorbed into the table as card 2 specifies, and the new `PATTERN-gogit-read-helper` background cites the renamed test.
  I found no duplicate test.

### [LOW] No test pins the HeadContains slack

**Location:** `internal/gitrepo/blobread_integration_test.go:233`
**Issue:** The new step puts the old commits days before the target.
A slack of zero, or any slack under a day, therefore passes it, and no other test dates a descendant within 24 hours of the target.
The behavior card 1 adds, that a walk keeps going through commits up to 24 hours older than the target, is uncovered.
That is the part that protects a target on a mildly skewed-clock history.
**Fix:** Add a row in the same `TestHistoryReads` step, or a second fixture in it.
It has a HEAD whose ancestors carry committer dates a few hours before the target's, and asserts `HeadContains(target)` is true.

### [LOW] PATTERN-hub-store-gc entry understates who writes the keys

**Location:** `PATTERN.md:36`
**Issue:** The entry reads "only fabricengine's housekeeping at pair removal runs `gc` or writes those keys".
`Topology.Add` also writes both keys, through `disableStoreAutoGC`, which lives in `housekeeping.go`.
The background file states the invariant correctly: `housekeeping.go` is the only lyx code that does either.
The overview line, which agents get inlined, is the narrower and wrong one.
**Fix:** Word the entry as "only `internal/fabricengine/housekeeping.go` runs `gc` or writes those keys", or name both Add and Remove.
Keep the line short, as it sits near `MaxOverviewBytes`.
