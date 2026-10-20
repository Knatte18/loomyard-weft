---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/01-failed-push-record.md:34
    summary: the way forward for a failed push or a diverged code branch of the hub prime names lyx fabric push, which never pushes the prime's code side
    fix: >-
      In 01-failed-push-record.md make the way-forward helpers prime-aware.
      Add a `Prime bool` field to `FailedPush` (set by `pushClassified` from the predicate F3 fixes, and by `ReconcileOrigin` for the prime's warp branch)
      and a `prime bool` parameter to `codeDivergedWayForward` (update its Create glyph signature, which is the re-sign).
      For a code-side record, and for a diverged code branch, of the prime's branch, `FailedPushWayForward` and `codeDivergedWayForward` name `git merge origin/<branch>` in the prime worktree, settling any conflict and committing, then `git push` there;
      the `deferred`, `transport` and `auth` lines of a prime code record also end in `git push` in the prime.
      None of them names `lyx fabric push` for the prime's code branch, because `PushPairAnchored` pushes only the records side in the prime and reports the code branch as `code_push_skipped`.
      `git push` is not in PATTERN-no-denied-recovery's closed list.
      Add `TestFailedPushRecord` rows for a prime code record in each of `transport` and `not_fast_forward`,
      and a row in card 15's `TestReconcileOrigin_Judge` asserting that a diverged prime warp branch's `Summary` carries the prime form.
      The change is card text only; no target list changes.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/15-origin-reconcile-judge.md:73
    summary: reconcile stops of gone, dirty, busy, live-and-moved branches and of a Bolt or prime stop name no way forward, and nothing carries one to the wake notice
    fix: >-
      In 15-origin-reconcile-judge.md give `BranchOutcome` a `WayForward string` field that `judgeBranch` and card 16's moves fill for every state that stops an entry or stops the scope, and render it in `OriginReport.Summary`.
      Today `Summary` names a line only for rewound, rewritten, diverged, busy under a parked push and ended-busy.
      Add the missing lines, each a runnable command outside PATTERN-no-denied-recovery's closed list:
      gone, for an entry origin shows neither done nor removed, re-run `lyx cleanup --apply` once origin's board shows the entry done, and `lyx fabric remove <slug>` from the prime when the branch was deleted by hand;
      dirty, commit the named paths (`lyx fabric commit` for a weft path, `git commit` for a code path) or restore them with `git checkout -- <path>`, then re-run `lyx cleanup --apply`;
      busy on a write lock, re-run `lyx cleanup --apply` once the holder ends;
      a live pair whose branch origin moved past or diverged from, `lyx batten pause <slug>` then `lyx cleanup --apply`;
      a D3 stop on Bolt or the prime weft, the dirty line above or `recordsConflictWayForward` with `lyx board sync`.
      Card 23's `WakeRun` and card 20's refusal take their line from `BranchOutcome.WayForward`, never composing their own.
      Add one table row per state to `TestReconcileOrigin_Judge` or `TestReconcileOrigin_Moves` asserting the exact line.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/09-push-retry-and-record.md:62
    summary: pushClassified has no input that tells a task branch from the prime's code branch, so its gone check and its rewound hold-back have no decidable scope
    fix: >-
      In 09-push-retry-and-record.md name the one predicate and its input.
      `pushClassified` receives only `side` and `path`, and `PushWarpRebaseFreeAt(warpPath, opts)` has only a path, yet the card limits the rewound hold-back to a task branch and exempts the prime's code branch.
      State that a code-side push is a task branch exactly when the worktree at `path` is not the hub's prime (compare `path` with the prime's worktree path under `hubPath`),
      so Finalize's push of a non-prime parent pair is judged as a task branch and the prime's `main` is not.
      Add that the same predicate sets `FailedPush.Prime` (F1).
      Add a `TestPushFailureClasses` row pushing the prime's code branch after an origin rewind, which is pushed and records nothing.
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/09-push-retry-and-record.md:60
    summary: the gone-path backoff reuses the delays of gitrepo.PushWithRetry, which are unexported constants, and no card exports them
    fix: >-
      In 08-push-auth-class.md add the Create target `plan:internal/gitrepo#PushBackoffDelays` -> `func PushBackoffDelays() []time.Duration`, returning the doubling delays `PushWithRetry` sleeps between its attempts (derived from `pushBaseBackoff` and `pushMaxAttempts`, never a second literal), and say so in its Intent.
      In 09-push-retry-and-record.md add `plan:internal/gitrepo#PushBackoffDelays` to Uses and say the gone-path loop sleeps those delays through `SyncOptions.BackoffSleep`.
      `lyx loom validate-plan --overlay` over both edited cards reports no blocking finding the baseline lacks.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-push-retry-and-record.md:83
    summary: transport failures of the gone path are classed at once in one bullet and retried by the backoff in a test row, and that row's input cannot be reached apart from the ls-remote
    fix: >-
      In 09-push-retry-and-record.md decide one rule and state it once.
      The recommended rule follows D10: a transport failure of `branchGoneOnOrigin`'s ls-remote or of the Bolt fetch is retried by the same `BackoffSleep` backoff, and only a failure still standing at its end writes the `transport` record;
      the bullet that classes any `gitexec.IsTransportFailure` error as `transport` at once then applies to the push call alone.
      The row `a gone branch whose Bolt fetch meets the refused URL` cannot be reached as written, because the weft's ls-remote and the Bolt fetch read one records repository and one `origin` URL,
      so the refused URL fails `branchGoneOnOrigin`'s ls-remote first, which is the upstream check that rejects the input.
      Replace the row with one whose ls-remote fails under a refused URL that the `opts.BackoffSleep` of the row restores, asserting no record until the backoff ends and a `transport` record when the sleep restores nothing,
      and drop the separate Bolt-fetch row, since both calls share the one handler the card names.
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/13-remove-local-mode.md:35
    summary: local removal keeps RemoveRefusal and removeSibling paths that run ls-remote, so the stated no-network guarantee fails for a pair without a local sibling and an ended entry without a pair is never handled
    fix: >-
      Two text changes.
      In 17-origin-reconcile-ended.md state that `reconcileEnded` calls `OriginSeams.Teardown` only when a local part of the pair exists (task worktree path, sibling worktree path, or either local branch), and otherwise reports the entry ended with no teardown, no error and no network call;
      a second machine's run record stays on the prime weft until the landing machine's cleanup reaches it, so such entries recur at every reconcile.
      Today `Topology.RemoveRefusal` runs `refuseWhenNothingRemains`, which calls `remoteHeadTip` for each side and then returns `ErrPairNotFound`, and the seam would report that as an error outcome on every reconcile.
      In 13-remove-local-mode.md state that `removePair` in local mode never takes `removeSibling`'s branch for a pair with neither sibling worktree nor sibling branch, which calls `remoteHeadTip`, and handles that case locally.
      Add a `TestReconcileOrigin_Ended` row for an ended entry with a run record and no local pair (no seam call, ended, no error, the call hook still counting two ls-remote calls),
      and a `TestRemoveLocal` row for a pair with a task worktree and no sibling, run with the records origin URL set to `http://127.0.0.1:1/x`, which removes without a remote error.
  - id: F7
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-land-weft-stop.md:19
    summary: the Land pull closure maps a weft stop to a nil error, which would also swallow a warp pull failure that card 6 joins to the stop
    fix: >-
      In 07-land-weft-stop.md add the Create target `plan:internal/battencli#pullStopOutcome` -> `func pullStopOutcome(err error) (recordsStop string, pullErr error)`, and have `landDeps`' Pull closure return its two results.
      The function returns a stop message with a nil error only when `err` is exactly a `*fabricengine.WeftStopError` with no warp failure joined to it, and returns any joined or other error as `pullErr`, so a warp pull failure stays the Land row's `pull` Stuck and the deploy does not run over an unpulled prime.
      Card 6 states that `Pull` joins the weft stop and the warp partial error (`TestPull_WeftDivergedAndWarpFetchFails_PartialError`), and an `errors.As` over such a join finds the stop and would drop the warp error.
      Add a table test of `pullStopOutcome` to card 7 with rows for a lone stop, a joined stop and warp error, and an unrelated error.
      The member name avoids the bare fabric side token that `fabric-vocabulary-name` refuses in battencli;
      `lyx loom validate-plan --overlay` over the edited card, with cards 8 and 9 as F4 and F8 edit them, reports no blocking finding the baseline lacks.
  - id: F8
    severity: BLOCKING
    class: scope
    location: _lyx/plan/09-push-retry-and-record.md:72
    summary: a failed push surfaced through commit-records, Publish or Finalize names lyx fabric push or push it by hand for gone, rewound and auth classes that refuse the same push again
    fix: >-
      In 09-push-retry-and-record.md specify `PushFailedError.Error()` as the push's own error followed by `; way forward: ` and `FailedPushWayForward` of the record,
      so every caller that prints the error carries its class's line.
      Add `internal/loomcli#pushFailureWayForward` and `internal/loomcli#TestCommitRecordsVerb_ErrorEnvelopes` to the Edit list:
      when `errors.As` finds a `*fabricengine.PushFailedError`, `pushFailureWayForward` returns `FailedPushWayForward(record)`, and only an error with none keeps today's text.
      Today it answers `transient, run lyx fabric push or re-run lyx loom commit-records` for any error that is not a rejection, which repeats the same stop for a `gone` branch (never pushed), a `rewound` task branch (held back) and an `auth` refusal.
      Add rows to `TestCommitRecordsVerb_ErrorEnvelopes` for `gone`, `rewound` and `auth` asserting each exact line,
      and a row to `TestPushFailureClasses` through `Fabric.PushBranch` asserting the returned error's text ends in the rewound line, since Publish's Stuck reads `push failed: <error>`.
      `lyx loom validate-plan --overlay` over card 9 with these two Edit targets, and with cards 7 and 8 as F4 and F7 edit them, reports no blocking finding the baseline lacks.
  - id: F9
    severity: MEDIUM
    class: scope
    location: _lyx/plan/20-cleanup-reconcile.md:21
    summary: the prescribed refusal of lyx cleanup apply on a reconcile error has no test row that reaches it
    fix: >-
      In 20-cleanup-reconcile.md add a step to the scenario test, with an input card 15 already defines:
      the records repo's origin URL set to a path that holds no repository, which `ReconcileOrigin` returns as an error because it is not a transport failure.
      `--apply` then refuses naming the error and runs no segment, and the dry run prints the error and still runs the segments.
      The `http://127.0.0.1:1/x` step in the same card reaches only the `Unreachable` warning, not this refusal.
  - id: F10
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/15-origin-reconcile-judge.md:59
    summary: pair removals and snapshot tags done through the teardown seam never reach the reconcile's mutation record, against PATTERN-mutation-record
    fix: >-
      In 15-origin-reconcile-judge.md give `TeardownReport` a `Mutations MutationRecord` field filled from the seam's `RemoveResult`, and in 18-reconcile-seams.md say `OriginSeams`' `Teardown` fills it.
      In 17-origin-reconcile-ended.md say `reconcileEnded` appends each report's mutations to the report's record (`rec.Extend`), so a removed pair, its tags and its deleted branches appear in the `mutations` array of orch start, cleanup and wake.
      In 13-remove-local-mode.md give `Topology.SnapshotUncommitted` a `rec *Mutations` parameter (update its Create glyph signature, which is the re-sign) and record the commit and the annotated tag it creates, so `RemovePair`'s result carries them.
      Add a `TestReconcileOrigin_Ended` assertion that the recording teardown fake's returned mutation appears in the `OriginReport`'s record,
      and a `TestRemoveLocal` assertion that the snapshot's tag appears in the mutation record.
  - id: F11
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/17-origin-reconcile-ended.md:19
    summary: finalizeEntered reads the local parent tip, which hub scope fast-forwards before reconcileEnded runs, against D6's judged before any move
    fix: >-
      In 16-origin-reconcile-moves.md and 17-origin-reconcile-ended.md pin the evaluation point:
      the reconcile calls `finalizeEntered` for every candidate slug once, right after `readEndedSet` and before Bolt's replay and any move, and passes the filtered `EndedSet` to every replay and to `reconcileEnded`.
      As written, `reconcileEnded` runs in D3 step 4 after the prime's branches moved, so its `HoldsWork` read sees a parent already fast-forwarded to the other machine's landing, and a run that never merged here reads as having entered Finalize and is neither set aside nor torn down.
      D6 says the judgment is made from the local records before any move.
      Add a `TestReconcileOrigin_Ended` row in hub scope: a pair status with `current_producer` `Finalize` and no history row, its entry done on origin, the local parent behind origin, is ended and torn down although the same reconcile fast-forwards the parent.
---

### [BLOCKING] Prime code branch way forward names a verb that skips the prime's code side

**Location:** `_lyx/plan/01-failed-push-record.md:34`, `_lyx/plan/15-origin-reconcile-judge.md:73`
**Issue:** Card 1 gives `lyx fabric push` in the branch's worktree as the way forward for `deferred`, `transport` and `auth`, and `codeDivergedWayForward` ends in it.
Finalize's parent push reaches `pushClassified` as a code-side push through `Fabric.PushBranch`, and for a hub whose parent is the prime that push is of the prime's `main`; the reconcile's hub scope also judges the prime's warp branch and can read it diverged.
`PushPairAnchored` in the prime pushes the records side only and reports the code branch as `code_push_skipped` (`internal/fabricengine/pushanchored.go`), so the named command leaves the record in place and the stop unchanged.
The focus directive asks whether each named command can push what it is named for; for this branch it cannot.

### [BLOCKING] Reconcile stops with no way forward

**Location:** `_lyx/plan/15-origin-reconcile-judge.md:73`, `_lyx/plan/23-wake-reconcile.md:34`
**Issue:** The decision record's constraints say every stop, refusal and failed-push class names an allowed way forward and a test reaches each.
Card 15's `Summary` names a line for rewound, rewritten, diverged, busy under a parked push and ended-busy only.
A gone branch of a not-ended entry, a dirty branch (including the dirty Bolt or prime stop that D3 makes a scope stop), a pair busy on a write lock, and a live pair origin moved past each stop an entry, and card 15 defines no field that carries a line for them.
Card 23's wake notice names "the stop's way forward" per stopped run, and card 20's refusal names "the stop's way forward", so both read an empty line for these states.
No test row of card 15 or 16 asserts a line for them (card 16 asserts only that a dirty branch stops with nothing moved).

### [MEDIUM] pushClassified cannot tell a task branch from the prime's code branch

**Location:** `_lyx/plan/09-push-retry-and-record.md:62`
**Issue:** The gone check runs for "a weft or a task branch with an upstream" and the rewound hold-back for "a task branch", with the prime's code branch exempt in the Bound line.
The signature `pushClassified(rec, hubPath, side, path, opts, retryRecords)` carries `side` as `code`, `records` or `bolt` and a path, and `PushWarpRebaseFreeAt` has only a warp path.
Publish's push and Finalize's parent push both arrive as `code`, so an implementer must invent the discriminator; a wrong choice either holds back the prime's push or lets a rewound task branch restore the dropped commits.

### [MEDIUM] Backoff delays named in card 9 are unexported in gitrepo

**Location:** `_lyx/plan/09-push-retry-and-record.md:60`, `_lyx/plan/08-push-auth-class.md:3`
**Issue:** Card 9 retries the gone path with "the delays `gitrepo.PushWithRetry` uses".
In `internal/gitrepo/pushretry.go` those are the unexported `pushBaseBackoff` and `pushMaxAttempts`, and `PushWithRetry` itself retries only a `(failed)` or `cannot lock ref` refusal of a pushed ref, which the gone path's ls-remote and fetch are not.
Card 8 is the only card that edits `gitrepo`, and it edits `PushRebaseFree` and `TestPush` only, so the fabricengine card has nothing to call.

### [MEDIUM] Transport handling of the gone path disagrees with itself, and one row is unreachable

**Location:** `_lyx/plan/09-push-retry-and-record.md:61`, `_lyx/plan/09-push-retry-and-record.md:69`, `_lyx/plan/09-push-retry-and-record.md:83`
**Issue:** The gone-path bullet says a transport failure of either call is "the push's own transport failure", and the classification bullet says a `gitexec.IsTransportFailure` error is class `transport` at once, since `PushWithRetry` retries only `(failed)` and `cannot lock ref`.
The test row says a gone branch whose Bolt fetch meets the refused URL "is retried as a transport failure and records nothing until the backoff ends", which is the opposite.
The row's input is also not reachable alone: the weft's `branchGoneOnOrigin` ls-remote and Bolt's fetch use the one records repository and its one `origin` URL, so the refused URL fails the ls-remote first.
`_lyx/plan/15-origin-reconcile-judge.md`'s `originCallHook` could separate them but belongs to a later card.

### [MEDIUM] Local removal still reaches the network through RemoveRefusal and removeSibling

**Location:** `_lyx/plan/13-remove-local-mode.md:35`, `_lyx/plan/17-origin-reconcile-ended.md:23`
**Issue:** Card 13 states local mode runs "no origin step and no ls-remote either" while keeping "every `RemoveRefusal` check".
`Topology.RemoveRefusal` calls `refuseWhenNothingRemains`, which asks origin for each side when nothing of the pair is local, and `removeSibling` calls `remoteHeadTip` when neither a sibling worktree nor a sibling branch exists; both are in `internal/fabricengine/remove.go`.
D6's bound says the local mode skips only origin steps and waits only for its own loop kill.
Card 17 hands every ended entry with a run record to the seam and says nothing for an entry whose pair is already gone, which is the state a second machine reaches after the first removal while the landing machine's run record is still on the prime weft.
The seam would then make two network calls beyond D1's two and report `ErrPairNotFound` as an error outcome at every reconcile.

### [MEDIUM] Land pull closure can hide a warp pull failure

**Location:** `_lyx/plan/07-land-weft-stop.md:19`, `_lyx/plan/06-pull-replay-reflog.md:47`
**Issue:** Card 7 has the closure return a `*fabricengine.WeftStopError`'s message as `weftStop` with a nil error and every other pull error as the error.
Card 6's `TestPull_WeftDivergedAndWarpFetchFails_PartialError` makes `Pull` return the weft stop joined with the warp partial error.
A closure that finds the stop by `errors.As` returns nil for that join, and `pullAndDeploy` then runs the deploy over a prime whose code failed to pull, where today the `pull` step is Stuck.
A direct type assertion would route the join to the error path, but the card does not say which, and no test reaches the closure.

### [BLOCKING] Failed-push classes surface through callers with a way forward that repeats the stop

**Location:** `_lyx/plan/09-push-retry-and-record.md:72`, `_lyx/plan/09-push-retry-and-record.md:73`
**Issue:** `PushFailedError.Error()` is unspecified, and the card says Publish's, Finalize's and commit-records' existing handling stays unchanged.
`internal/loomcli/commitrecords.go`'s `pushFailureWayForward` answers `transient, run lyx fabric push or re-run lyx loom commit-records` for any error that is not a rejection; the `gone`, `rewound` and `auth` classes all fall there, and each refuses the same push again, a gone branch being never pushed and a rewound task branch held back.
Publish's Stuck reason is `push failed: <error>` and Finalize's is `push it by hand`, so neither carries the class's line either.
The decision record's constraint requires each failed-push class to name an allowed way forward and a test to reach it; card 1's test reaches the helper's lines, not the callers that print the stop.

### [MEDIUM] Cleanup's refusal on a reconcile error has no reaching test

**Location:** `_lyx/plan/20-cleanup-reconcile.md:21`, `_lyx/plan/20-cleanup-reconcile.md:26`
**Issue:** The card prescribes that a reconcile error refuses `--apply` naming it and runs no segment, and that the dry run prints it and runs the segments.
The scenario test's rows reach a D3 stop, an `Unreachable` report and a pair stop, never an error return of `ReconcileOrigin`.
The refused-URL input the card uses yields `Unreachable`, not an error, so it does not stand in; card 15 already names the input that does, an origin URL naming a path that holds no repository.

### [BLOCKING] Pair removals bypass the reconcile's mutation record

**Location:** `_lyx/plan/15-origin-reconcile-judge.md:59`, `_lyx/plan/13-remove-local-mode.md:7`
**Issue:** PATTERN-mutation-record, and the decision record's constraint on it, require the reconcile to fill a `*Mutations` record and every mutating result to expose it.
`OriginReport` embeds `MutationRecord`, but a pair's removal runs through the injected seam and comes back as a `TeardownReport` that carries tags, kept reasons, a busy flag and an error only, so the largest mutation of the reconcile never reaches the `mutations` array of orch start, cleanup or wake.
`Topology.SnapshotUncommitted(l, slug)` also creates a commit and an annotated tag and takes no record, so the snapshot a force-case removal reports never enters `RemoveResult`'s record either.

### [MEDIUM] finalizeEntered is read after the reconcile moved the parent

**Location:** `_lyx/plan/17-origin-reconcile-ended.md:19`, `_lyx/plan/16-origin-reconcile-moves.md:21`
**Issue:** Card 4 reads whether this machine's own landing has run through `Fabric.HoldsWork(<local parent tip>, <task head>)`, and D6 says ended is judged "from the local records before any move".
Card 16 moves Bolt, then the prime's branches, then each pair, and card 17 runs `reconcileEnded` in D3 step 4 after the prime's branches; the plan does not say when `finalizeEntered` is evaluated.
Read at `reconcileEnded`, the prime's local `main` has been fast-forwarded to origin, so a pair that stopped at `Finalize` and never merged here reads as having landed here, and the entry another machine landed is not set aside or torn down.
Run scope never moves the parent, so the Merge-In and Finalize halts card 22 prescribes are unaffected.

## Rejected

- Focus entry 1, `finalizeEntered` and the ended-at-Finalize Halt: card 4's rule counts `current_producer` `Finalize` only with the local parent holding the task's work (`Fabric.HoldsWork` handles a squash landing by tree equality) or a Finalize history row or a prime batten status at `Land`, so a run another machine landed reads ended at run scope, where the parent is never moved; card 17's `TestReconcileOrigin_Ended` rows reach it over pair status files. The hub-scope ordering defect is F11.
- Focus entry 3, `LandDeps.Pull` retype: the four test literals the card names are every literal in the tree (`internal/battenshed/land_test.go`, `internal/battencli/run_test.go`, `internal/shedrecipe/fixture_test.go`, `internal/testkit/envkit/envkit.go`), each in the named function, and every package is in the `## verify:` section.
- Focus entry 4, injectable deadline and sleep: `ReconcileOptions` and `gitexec.RunWithin` take the deadline as a parameter, `SyncOptions.BackoffSleep` is a field read at call time, `HubPaused` is read at each `LivePair` call, and `gitexec` and `pauserecord` are in the verify section; the gate wait still precedes the deadline in `runCore`. The one gap is F4's unexported delays.
- Focus entry 5, push paths: `Fabric.PushWeft` and `PushWarpAt` have no production caller (only tests call them), so card 9's explicit exclusion is accurate; `PushWarpRebaseFreeAt` is reached by Publish and Finalize through `Fabric.PushBranch`, and card 9 covers it with a `Fabric.PushBranch` row. The deferred `respawnPairPush` is skipped in local mode by card 13; the remaining network reach of local mode is F6.
- Card 11's premise holds: `hubreconcile.reconcileHubWide`'s `bolt.Push` is the only synchronous records push the Preflight row's `Ensure` makes.
