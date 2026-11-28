```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the set found and whether it is covered:**

1. **`worktreeGone(path)`: new signature, `.git`-entry definition, wrapped stat error.**
   - Callers: `TargetOf`, `TargetRefused`, `PruneLedger`, the starter's `pairGone`, `NewDeps`'s `WorktreeGone`. All are in the diff. Covered.
   - Test: `seams_test.go` `TestWorktreeGone_ReadsWhetherTheWorktreeStands`. Covered.
   - Sibling "pair stands" read in the same package, `parentseams.go:59-70` `pairOf`, still decides on the directory alone. Not covered: finding 1.
   - Field doc `run.go:58`. Not updated: finding 2.
2. **`TargetOf` returns a pair target with nil error for a gone worktree.**
   - `battencli/commitstatus.go:170,204`, `loomcli/wiring.go:216,1413`: each passes the target to `Start`, which skips silently. Covered.
   - `loomcli/pusher.go:78`: `lymcwd.ResolveWorktree` runs first and fails on a gone path. A leftover directory gets a pair target, then `Run`'s `canPush` sees the worktree as gone and exits. Covered.
   - Tests: `adoptcli/adopt_integration_test.go:153` and `loomcli/pusher_integration_test.go:51` use real worktrees. Unaffected.
3. **`TargetRefused` answers false for a gone pair.**
   - `beamcli/dobbydeps.go:467` `settleIfRefused`: a gone pair is not settled and goes on to a start that skips. Consistent.
   - `beamcli/dobbydeps.go:560` `refusedPushWayForward`: its doc already says "a pair whose worktree is gone reads as not refused". A leftover directory now matches that doc as well. Covered.
   - `loomcli/landingdeps.go:1042` `parentPushState` reads a parent target. A parent that no longer stands reads as not refused, which leaves `Owed` alone. No doc to update.
   - `run.go:88` `Conditions`. Covered.
4. **`Start`/`StartIfAhead` skip a gone pair before any other read.**
   - Callers: `battencli/dobbyreads.go:281,323` (`DobbyRuns` via `restartPusher`, only when `PairPresent`), `beamcli/ownerarchive.go:33`, `dobbydeps.go:259,267`, `landingdeps.go:940`, and `parentseams.go` `WakePair`. All take the silent skip.
   - Integration tests over real hubforge pairs (`beamcli/dobbypairanchor_test.go`, `battencli/dobbyruns_integration_test.go`, `teardownpush_integration_test.go`, `catchuppush_integration_test.go`) build pairs with `.git`. Unaffected.
   - Doc: `doc.go` and the `Start` comment are updated. Covered.
5. **`PruneLedger` drops a leftover directory's entry.**
   - Caller: `dobbydeps.go:270`, whose doc says "drops the push ledger entries of gone worktrees". Generic wording, still true.
   - Readers of the ledger: `hubPushSummary`, `parentTargets`. An entry that holds landings, dropped commits or owed hand-overs is kept by the landings guard only, so for a leftover directory the `Dropped`/`HandOvers` parent work goes. This is a behaviour question for the correctness lens; I raise no affected-set finding on it.
   - Rule docs: `pattern/PATTERN-pair-remote.md`, `PATTERN.md` and `docs/overview.md` hold no text on ledger pruning or a pair's definition of gone. Nothing to update.
6. **Commit messages and departures.**
   - `f60f0a32` carries a `Root-Cause:` trailer naming the directory-only presence read and the git reads on a removed path, which the diff removes.
   - There is no `Pattern-Departure:` or `Loosened-Check:` trailer. The diff loosens no existing assertion: the `TestPruneLedger` standing fixture gained a `.git` entry, which tightens it.
   - No declared PATTERN changes. No stencil touched. Nothing to flag.

**Findings:**

1. **LOW · consistency · `internal/runpusher/parentseams.go:59-70` (`pairOf`)**
   - The parent seams' own "pair stands" read still decides on `os.Stat(pair.WorktreePath())`.
   - So a leftover directory without `.git` reads as standing there. `WorkUnderOtherCommit`, `UnpushedRecords` and `HoldPair` then run git reads on it (`parentseams.go:156-164,195-199,203-210`), and `HoldPair` writes a ledger entry for it.
   - Meanwhile `worktreeGone` in the same package reads that directory as gone.
   - This contradicts the change description's "A pair stands only while `<worktree>/.git` exists" (`_lym/landing/summary.md:16`) for the package the summary says all of the change is in.
   - Evidence: `parentseams.go:59-70`, `targetreads.go:86-95`.
2. **LOW · consistency · `internal/runpusher/run.go:58`**
   - The `WorktreeGone` field doc still reads "reports whether the worktree no longer exists".
   - The diff rewired it (`run.go:98`) to `worktreeGone`, which also answers true for a directory that still exists without its `.git` entry.
   - The doc now misstates the seam's contract. Compare `targetreads.go:84-85`.

**Tool calls:**
1. grep for `_test.go` callers of the changed runpusher exports, and for repository docs naming ledger pruning or a gone worktree: the manifest held production callers only, no docs search.
2. grep of those test files' call lines and the overview doc's pusher lines: needed to see which tests reach the changed reads.
3. Read of `beamcli/dobbypairanchor_test.go:60-240`: to check whether its pair fixtures lack `.git` and depend on the old behaviour.
4. grep for `pairOf` uses in runpusher: the manifest had its definition but not its uses.
5. Read of `runpusher/parentseams.go:150-245`: to see what each `pairOf` caller does with a standing leftover directory.
