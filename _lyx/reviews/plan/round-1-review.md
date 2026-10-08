---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/05-hub-store-gc.md (Tests, fourth subtest)
    summary: A held gc.pid makes git gc --auto exit 0 silently, so the planned gc-failure subtest can never see the warning it asserts
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/02-gitrepo-retrying-read.md (Tests, parity_test.go)
    summary: The #468 regression table shares one stale-pack fixture across rows, so every row after the first reads an already-reindexed handle and passes vacuously
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/04-gitexec-remote-bounds.md (Intent, logging exemption paragraph)
    summary: The decision record's PATTERN-spawn-observability constraint that a timed-out git kill is logged through internal/logger is carried by no card; card 4 discards the kill outcome
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-gitexec-remote-bounds.md (Tests, transport_test.go)
    summary: Card 4 adds a new pull-with-transport-phrase row while the existing pull with phrase row asserts false for the same input; that row must flip instead
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/01-headcontains-bounded-walk.md (Intent, test fixture)
    summary: go-git's committer-time walker loads each returned commit's parents, so deleting the first or second commit past the bound breaks the walk; the fixture does not say which commit to delete
  - id: F6
    severity: LOW
    class: design
    location: _lyx/plan/04-gitexec-remote-bounds.md (Tests, gitexec_test.go)
    summary: A single proc.IsAlive check right after the kill can see the killed ext transport child as a not-yet-reaped zombie; the test should poll to a bound
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/03-proc-group-kill.md (Intent)
    summary: verifyrun's package doc says it imports only the standard library and internal/logger, which card 3 makes false without updating it
  - id: F8
    severity: LOW
    class: scope
    location: _lyx/plan/02-gitrepo-retrying-read.md (Tests)
    summary: parity_test.go's forcePackIndexFreeze doc still points at gogit.go's lookupObjectRetrying, which card 2 deletes; only gogit_test.go's file comment is planned for update
---

### [BLOCKING] Card 5's gc-failure subtest relies on a failure git never reports

**Location:** `_lyx/plan/05-hub-store-gc.md`, Tests, the subtest "With `gc.pid` held in a store by a live process on this host, a Remove still succeeds and the captured log holds a warning naming the gc failure".

**Issue:** When another live gc holds `gc.pid`, `git gc --auto` exits 0 quietly and does nothing; only a non-`--auto` gc dies with "gc is already running".
I confirmed this against git 2.53.0 in a scratch repo: with `gc.pid` naming a live `sleep` on this host and `-c gc.auto=1 -c gc.autoDetach=false -c gc.worktreePruneExpire=never gc --auto --quiet`, git exited 0, packed nothing, and printed nothing.
`housekeepStores` therefore sees success and logs no warning, so the subtest as specified can never pass.
The acceptance criterion it is meant to carry ("a gc failure leaves the removal successful with a logged warning") is left with no working test, and the card's Intent repeats the false premise ("each failure, including another gc holding `gc.pid`, is logged as a warning").

**Fix:** Drive the failure through a case where `git gc --auto` exits non-zero, e.g. a store whose `objects/pack` directory the test makes unwritable after lowering the threshold, or a `-c`-invalid config value in the store's config.
Correct the Intent: a held `gc.pid` is a silent skip, not a logged failure.

### [MEDIUM] Card 2's #468 regression rows share one stale fixture

**Location:** `_lyx/plan/02-gitrepo-retrying-read.md`, Tests, `parity_test.go`.

**Issue:** The card describes one fixture (freeze the pack index, CLI commit, `git repack -d`, `git prune-packed`) followed by "each row asserts one method answers correctly afterwards".
On one `Repo`, the first row's object-not-found triggers `readGoGit`'s reindex and updates `lastPackFingerprint`, so every later row reads a fresh index and passes even if its method bypasses `readGoGit`.
Only the first row would regress-test #468; the acceptance criteria name each of `UntrackedFiles`, `WorktreeChangedFiles`, `ChangedFilesSince`, `HeadContains`, `PathRevisions` and a merge-state read.
A fresh `gitrepo.New` per row on the repacked directory does not help either, since it opens a non-stale handle.

**Fix:** State that each row builds its own fixture and runs the freeze → commit → repack sequence itself before its one read, as `TestSHAExists_MixedBackend_RepackBetweenCommitAndRead` does today.

### [MEDIUM] The timed-out kill's logging constraint is carried by no card

**Location:** `_lyx/plan/04-gitexec-remote-bounds.md`, Intent, the paragraph beginning "`gitexec` cannot import `internal/logger`".

**Issue:** The decision record's Constraints list says "`PATTERN-spawn-observability`: the housekeeping gc and a timed-out git kill are logged through `internal/logger`".
Card 5 carries the gc half.
Card 4 makes the kill's `report` callback do nothing and says the timeout "reaches the logs through the `*GitError` every caller logs".
That claim is false in general: many remote call sites (`cleanupremotewarp.go`'s `ls-remote`s, `remoteleftover.go`, `add.go`'s fetches) return the error up to a CLI envelope, not to the logger.
The group-kill-failed outcome, which verifyrun logs as a warning because descendants may outlive the process, is not on the `*GitError` at all and is dropped.
The import-cycle reason is real, and the PATTERN's own exemption clause keeps this from being a PATTERN violation, but the plan departs from an answer-key constraint without recording the departure.

**Fix:** Either carry the constraint, e.g. by having a logger-importing package that every `lyx` command loads install the kill reporter on gitexec at startup, or record the departure with `lyx loom decision add` and drop the "every caller logs" claim from the Intent in favor of what actually reaches the logs.

### [MEDIUM] Card 4 adds a pull row beside an existing row asserting the opposite

**Location:** `_lyx/plan/04-gitexec-remote-bounds.md`, Tests, `transport_test.go`.

**Issue:** `internal/gitexec/transport_test.go`'s `TestIsTransportFailure` already has `{"pull with phrase", &gitexec.GitError{Args: []string{"pull"}, Stderr: "Connection refused"}, false}`.
Card 4 plans a new row "for a `pull` failing with a transport phrase … true".
That duplicates the existing row's input (`PATTERN-test-economy`), and if the existing row stays, the test fails once `pull` joins `remoteSubcommands`.

**Fix:** Flip the existing `pull with phrase` row's expectation to true instead of adding a row.
The `Operation too slow` phrase fits the existing `phrases` list that the test iterates over push and fetch, rather than a separate row.

### [LOW] Card 1's fixture leaves the deleted commit unspecified

**Location:** `_lyx/plan/01-headcontains-bounded-walk.md`, Intent, the test paragraph.

**Issue:** go-git's `commitIteratorByCTime.Next` calls `GetCommit` on every parent of the commit it returns before the `ForEach` callback sees that commit.
In the planned history (HEAD → old commits), the walk returns HEAD after loading the newest old commit, then returns that old commit, stopping in the callback, after loading its parent.
Deleting either of those two commits' object fails the bounded walk too, so "deletes one old commit's loose object file" passes only if the implementer picks a commit at least three generations below HEAD.

**Fix:** Name it: build at least three old commits and delete the oldest one's object, and say why the two nearest are read.

### [LOW] Card 4's liveness check can race the zombie reap

**Location:** `_lyx/plan/04-gitexec-remote-bounds.md`, Tests, `gitexec_test.go`, first row.

**Issue:** After the group kill, the `ext::` child sits as a zombie until init reaps it, and `proc.IsAlive` (signal 0) succeeds on a zombie.
A single check right after `Run` returns is not deterministic.
`verifyrun`'s `TestRun_ExpiryKillsBackgroundedDescendant` polls with a bounded deadline for the same reason.

**Fix:** Poll `proc.IsAlive` to a bounded deadline, as the verifyrun test does.

### [LOW] verifyrun's package doc goes stale

**Location:** `_lyx/plan/03-proc-group-kill.md`, Intent.

**Issue:** `internal/verifyrun/run.go`'s package doc says "It imports only the standard library and internal/logger."
After card 3, `verifyrun` imports `internal/proc`, and the card does not mention the doc.

**Fix:** Have the Intent update that sentence.
The doc is outside `run`'s span, so replace the `internal/verifyrun#run` target with the file `internal/verifyrun/run.go`, rather than adding the file beside the member, which `redundant-file-target` refuses.

### [LOW] A test-helper comment still names the deleted helper

**Location:** `_lyx/plan/02-gitrepo-retrying-read.md`, Tests.

**Issue:** `parity_test.go`'s `forcePackIndexFreeze` doc says the later read "can only resolve it correctly by going through the fingerprint-gated reindex retry (see gogit.go's lookupObjectRetrying)".
Card 2 deletes `lookupObjectRetrying` and plans the comment update only for `gogit_test.go`'s file comment.

**Fix:** Add to the Tests list: `forcePackIndexFreeze`'s doc names `readGoGit`'s whole-read retry.

## Verdicts on focus items with no finding

- Re-sign arrows: no card needs one.
  `runCore`, `RunGit` and `Run` keep their signatures (a timeout comes back through the existing `err` result); `GitError` is a struct whose new field takes no arrow; `Repo.goGit` changes only its doc; `checkRunStatus` is a `Create`.
- Card order: no card depends on a later one.
  Card 1 edits `HeadContains` before card 2 moves it, card 4 uses card 3's `proc.ConfigureGroupKill`, and card 5 uses room card 2 frees (`PATTERN.md` is 16185 of 16384 bytes).
- Card 2 test economy: the parity table absorbs `TestSHAExists_MixedBackend_RepackBetweenCommitAndRead`.
  The concurrent-callers row and the reindex-at-most-once subtest pin reindex counts no existing test asserts, and `gogitcaller_test.go` pins an invariant no existing scan covers.
- Card 4 test economy, apart from F4: the new integration test (deadline, kill, environment) covers nothing `TestIsTransportFailure_RealGit` does.
- Attack surface: cards 4 and 5 state their bounds (deadline and speed limit with worst case; store-wide auto-gc off, manual gc still working, admin entries never pruned), and card 6's refusal lets nothing new through.
- Fidelity: D1–D6 are each carried, including D3's `pull`, D4's shared `checkRunStatus`, and D6's `doc.go` updates for gitrepo, gitexec, fabricengine and battencli (in `cli.go`).
  Every constraint except F3's half is carried, including `PATTERN-no-denied-recovery` (card 6) and `PATTERN-hubforge-fixtures` (card 5).
  No card draws scope from the carry-over block.
- Verify coverage: `./...` under both run tiers covers every targeted package, and the section follows the template's order.
