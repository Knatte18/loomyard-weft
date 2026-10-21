---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/01-failed-push-record.md:43
    summary: "rewoundWayForward offers an ours merge that is a no-op when origin's head is already in the local tip's history, and card 9's hold-back refuses the lyx fabric push after it, so the keep-local choice can never settle a rewind"
    fix: "In card 1, rewrite the second choice of `rewoundWayForward` as `git push origin <branch>` in the task worktree, a plain fast-forward since origin's head is an ancestor of the local tip, which restores the commits the other machine dropped and writes the push entry that ends the reflog test, and drop the `git merge --strategy ours origin/<branch>` step and its claim that the merge makes the push a fast-forward. In card 9's Intent state why the ours merge is not offered (merging a commit already in the tip's history changes nothing, and the hold-back reads the tip as still rewound) and that `lyx fabric push` stays refused until the rewind is settled. In card 9's `TestPushFailureClasses` rows add two settle rows for a rewound task branch, one after `git reset --keep <origin head>` and one after the raw `git push origin <branch>`, each asserting the next `lyx fabric push` lands, removes the `rewound` record, and the next reconcile no longer reads the branch rewound."
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-records-replay.md:13
    summary: set-aside and snapshot tags have no mutation Kind, and no planned row asserts that the reconcile's fast-forwards and set-aside tags reach the OriginReport record that orch start and cleanup envelopes and the wake log read
    fix: "In card 4 add `- plan:internal/fabricengine#KindTagCreated` -> `const KindTagCreated Kind = \"tag_created\"` to Create (with the glyph in backticks as the other lines write it), an Edit section naming `internal/fabricengine_test#manifestObservableKind` (classified false, a git-state kind), and an ImpactSummary stating one new kind and one oracle row; say in the Intent that `setAsideTag` appends it only after the tag exists, Target the repo root, Detail the tag name, and in card 13's `SnapshotUncommitted` paragraph that the snapshot tag is appended the same way. In card 16's `TestReconcileOrigin_Moves` rows assert that the report's `Mutations.Entries()` holds a `repo_advanced` entry per fast-forward and a `tag_created` entry per set-aside tag of a replay, and holds none for the judge-only scope. This overlay (card 4 edited as described) passes `lyx loom validate-plan --overlay` with no finding beyond the baseline."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/15-origin-reconcile-judge.md:78
    summary: the dirty line names `lyx fabric commit` for a weft path with no directory, and the rewritten line names `lyx fabric pull` in the pair's worktree for every branch, so from the prime each command acts on the prime instead of the pair, and the exact-line rows pin no directory
    fix: "In card 15 give each line its directory. Dirty weft path of a pair, run `lyx fabric commit` in the pair's task worktree, since every fabric verb is refused in the weft sibling; dirty weft path of the prime weft, `lyx fabric commit` in the prime worktree; dirty code path, `git commit` or `git checkout -- <path>` in the pair's task worktree or the prime worktree as the branch belongs; Bolt path unchanged. Rewritten, `lyx fabric pull` in the pair's task worktree, or in the prime worktree for the prime's warp branch. In card 16's rows pin each of these lines exactly (a pair weft, the prime weft, a code branch) and in `TestReconcileOrigin_Judge` replace the rewritten row's `names lyx fabric pull` with the exact line for a pair and for the prime."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/15-origin-reconcile-judge.md:78
    summary: BranchOutcome.WayForward is one string every hop prints, yet the dirty and gone lines end in the hub-scope re-entry `lyx cleanup --apply`, which the Merge-In, Finalize, batten resume and wake hops append their own verb to, so a run-scope stop prints a verb cleanup refuses outside the prime and the wake notice prints two re-entries
    fix: "In card 15 make `BranchOutcome.WayForward` the repair step alone for dirty, gone and busy (commit or restore the paths, wait for the holder, `lyx fabric remove <slug>` from the prime) and drop `then re-run lyx cleanup --apply` from those lines. Each hop appends its own re-entry and its row pins the whole text: the cleanup refusal and the orch start summary append `lyx cleanup --apply` from the prime, card 21's refusal appends `lyx batten run <slug>`, card 22's Stuck keeps `lyx loom resume` in the task worktree, card 23's notice keeps `lyx resume <slug>` for a stopped pair and `lyx cleanup --apply` for an ended busy one. Adjust the exact-line rows of cards 15, 16, 20, 21, 22 and 23 to the split."
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/19-orch-start-reconcile.md:28
    summary: the orch start, batten resume and Merge-In and Finalize rows assert that a stop is named, not the exact line the judge produced, so a hop that composed its own line would pass
    fix: "In card 19's third row give the fake report a stop whose `BranchOutcome.WayForward` is a distinctive sentinel and assert the start prompt's added line contains `OriginReport.Summary()` of that report verbatim. In card 21's `TestRunCmd_ResumeDispositions` stopped row assert the refusal contains the fake report's `WayForward` verbatim followed by the hop's re-entry. In card 22's stopped rows of `TestMergeIn_RunsStatusCommitThenLockThenSyncThenMergeAndVerify` and `TestFinalize_StuckReasons` assert the Stuck reason contains the fake report's `WayForward` verbatim followed by `lyx loom resume`."
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/01-failed-push-record.md:48
    summary: no planned row asserts the text of the failed-push mail, so a RecordPushFailure that dropped or reworded the class's way-forward line would still pass every row that counts mails
    fix: "In card 1's `TestFailedPushRecord` first-failure row make the fake notifier record the line it receives and assert it equals the composed line, naming the branch, the class, the error line and `FailedPushWayForward` of the record exactly. In card 2's `TestInstallPushNotifier` assert the posted mail's text equals the line `RecordPushFailure` passed."
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-push-retry-and-record.md:85
    summary: Publish's ErrPushRejected branch returns pushRejectedReason before the push failed Stuck, and a not_fast_forward PushFailedError wraps that sentinel, so the class line card 9 says Publish prints is never printed for the one class Publish most often meets, and no row covers it
    fix: "In card 9 state that Publish keeps `pushRejectedReason` for a `*PushFailedError` of class `not_fast_forward` (its remote-only-commits detail is richer than the class line) and appends nothing to it, and that the other classes reach the `push failed:` reason carrying the class line. Add a `not_fast_forward` row and a `transport` row to the `TestPublish_StuckBeforePullRequest` rows card 9 already adds, asserting respectively that the reason is `pushRejectedReason`'s text with no second way-forward line and that the reason ends in the class's exact line."
  - id: F8
    severity: MEDIUM
    class: scope
    location: _lyx/plan/19-orch-start-reconcile.md:22
    summary: the card gives orch start's error envelope after a mutating reconcile a partial key, but every error path in startCmd goes through the bare fail closure, and no row asserts an error envelope carrying the record
    fix: "In card 19's Intent say that once the reconcile has run, `startCmd`'s `fail` closure builds the error envelope with the report's `mutations` and `partial` (true when the record is non-empty), and that errors before the reconcile stay bare. Add a row to `TestStart_ReconcilesTheHubFirst` whose fake report carries one teardown mutation and whose launch then fails, asserting the error envelope holds that mutation in `mutations` and `partial` true, and a row whose launch fails after an empty-record reconcile asserting `mutations` an empty array and `partial` false."
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/09-push-retry-and-record.md:89
    summary: reconcileFailureWayForward is appended after a PushFailedError text that already ends in the class's line, so a failed board push prints two way-forward lines, and no row covers the reworded text
    fix: "In card 9 make the fabric reconcile verb append its `re-run lyx fabric reconcile` line only when the failure is not a `*PushFailedError`, by the rule card 9 gives `commitRecordsVerb`, and add a fabriccli row asserting a `*PushFailedError` of a board push prints its class line exactly once and a commit failure prints the reconcile line once."
---

### [BLOCKING] The keep-local rewind choice cannot be carried out

**Location:** `_lyx/plan/01-failed-push-record.md:43`, `_lyx/plan/09-push-retry-and-record.md:69`

**Issue:** `rewoundWayForward` offers two choices. The first, `git reset --keep <origin head>` then `lyx fabric push`, works: the tip equals origin's head, so the branch no longer reads rewound. The second, `git merge --strategy ours origin/<branch>` then `lyx fabric push`, is claimed to keep the local content and make the push a fast-forward. A branch is rewound only when origin's head is already an ancestor of the local tip, so that merge reports it is up to date and creates no commit, and the push was already a fast-forward. Card 9's hold-back then reads the same tip as still rewound: an earlier reflog value is not an ancestor of origin's head, origin's head is an ancestor of it, and the tip holds that value's commits. It refuses the `lyx fabric push` the line names, and only a push writes the `update by push` entry that ends the reflog test, so the choice loops. No planned row settles a rewind and then pushes, so nothing reaches the loop. The answer key's D4 names the `ours` merge, but the plan owns the step it prescribes; a step whose result the plan's own hold-back refuses is a defect of the plan.

### [MEDIUM] Reconcile moves and tags have no record kind and no record assertion

**Location:** `_lyx/plan/04-records-replay.md:13`, `_lyx/plan/13-remove-local-mode.md:41`, `_lyx/plan/16-origin-reconcile-moves.md:36`

**Issue:** `Kind` in `internal/fabricengine/mutation.go` is the single declarer of the mutation kinds, and its comment says a new member lands with its recording site; `livestate_mutationoracle_test.go`'s `manifestObservableKind` classifies every kind so a new one fails loud. Cards 4 and 13 record set-aside and snapshot tags in `rec`, yet no existing kind names a local tag (`KindTagPushed` is a pushed archive tag), and no card adds one or names the kind used. For the focus question on orch start, `lyx cleanup --apply` and the wake: card 17 asserts the teardown fake's mutation reaches the `OriginReport` record and cards 19, 20 and 23 carry it to the envelopes and the log, but card 16's rows say a fast-forward is "reported" without asserting an entry in `Mutations`, and no row asserts a replay's set-aside tag there. The moved branches and the tags are two of the three things the focus names.

### [MEDIUM] Way-forward lines name no directory for the fabric commit and pull steps

**Location:** `_lyx/plan/15-origin-reconcile-judge.md:75`, `_lyx/plan/15-origin-reconcile-judge.md:78`

**Issue:** `RequireWarpWorktree` refuses every fabric verb in a weft sibling, and `lyx fabric commit` and `lyx fabric pull` act on the pair of the worktree they run in. The dirty line says `lyx fabric commit` for a weft path with no directory, while a hub-scope stop is printed to an operator standing in the prime, where the command would commit the prime's weft, not the pair's. The rewritten line says `lyx fabric pull` "in the pair's worktree" although the rewritten state applies to the prime's warp branch too. The rows pin "the dirty line exactly" and "names `lyx fabric pull`" without the plan fixing what directory the line names, so a line with the wrong directory satisfies them. Card 1's lines do name their directory and pin it, which is the form these two need. The `_board` dirty form (`git commit` in the hub's `_board` worktree) is runnable and pinned by cards 16 and 20.

### [MEDIUM] One way-forward string carries a re-entry verb that is wrong at three hops

**Location:** `_lyx/plan/15-origin-reconcile-judge.md:78`, `_lyx/plan/22-landing-reconcile.md:24`, `_lyx/plan/23-wake-reconcile.md:39`

**Issue:** Cards 20, 21, 22 and 23 print `BranchOutcome.WayForward` "and never compose their own", and card 15's dirty line ends `then re-run lyx cleanup --apply`, its gone line `re-run lyx cleanup --apply once origin's board shows the entry done or removed`. At Merge-In and Finalize the stop is printed in the task worktree, where `lyx cleanup` refuses, and card 22 then appends `lyx loom resume`, so the Stuck names two re-entries. At the wake, card 23 appends `lyx resume <slug>` to the same line, which tells the operator to re-run a hub cleanup the pair's own run-scope reconcile makes unnecessary. Batten resume appends nothing, so its refusal has no re-entry of its own. The repair step belongs to the judge and the re-entry to the hop.

### [MEDIUM] Three printing hops assert a named stop, not the judge's line

**Location:** `_lyx/plan/19-orch-start-reconcile.md:28`, `_lyx/plan/21-batten-resume-reconcile.md:28`, `_lyx/plan/22-landing-reconcile.md:30`

**Issue:** The focus asks that a test at each hop assert the exact line the judge produced, so a hop that composed its own line fails. The wake rows do (the fake `WakeRun`'s exact line), card 20's rows do for Bolt and the prime weft, and card 6's pull rows do. Orch start's row asserts "the stop in the start prompt's summary line", batten resume's "naming the branch and way forward", and Merge-In's and Finalize's "naming the branch, its way forward and `lyx loom resume`". Each hop is a separate site with its own assembly of the line, so a substring check on the branch name leaves a composed line unfailed.

### [LOW] The failed-push mail text is never asserted

**Location:** `_lyx/plan/01-failed-push-record.md:48`, `_lyx/plan/02-push-notifier-install.md:23`

**Issue:** The mail is the one place the class's way-forward line reaches the session that acts on a push failure. Card 1's rows count notifier calls and check `notified`, card 2's row checks one mail signed `FABRIC`, and card 9's `TestPushFailureClasses` counts mails. Card 1 specifies the line as naming the branch, the class, the error line and `FailedPushWayForward`, but no row compares what the notifier received with that composition.

### [MEDIUM] Publish never prints the class line for a code-side rejection

**Location:** `_lyx/plan/09-push-retry-and-record.md:83`, `_lyx/plan/09-push-retry-and-record.md:85`

**Issue:** `internal/landingshed/publish.go` checks `errors.Is(err, gitrepo.ErrPushRejected)` before its `push failed:` return and builds `pushRejectedReason` for it. Card 9 makes a `not_fast_forward` `*PushFailedError` wrap that sentinel "so `IsPushRejected` callers keep working", so Publish takes the rejection branch for it, and card 9's sentence that Publish's `push failed: <error>` Stuck prints the class's line holds for `gone`, `rewound` and `auth` only. The new rows cover those three and leave the existing rejection rows as they are, so the card neither says which line Publish prints for `not_fast_forward` nor pins it. This is not a finding about the prime form: Publish pushes a task branch, so the prime flag does not arise there, and card 9's `Fabric.PushBranch` rows reach it from the prime for both a rejection and an 403.

### [MEDIUM] Orch start's error envelope after a mutating reconcile is unspecified and untested

**Location:** `_lyx/plan/19-orch-start-reconcile.md:22`, `_lyx/plan/19-orch-start-reconcile.md:28`

**Issue:** `PATTERN-mutation-record` gives a pre-flight failure a bare `output.Err` and a mutating verb's envelope `mutations` and `partial`. Card 19 puts the reconcile after the start lock and defines `partial` as "true exactly when the envelope is an error and the record is non-empty", but every failure path in `startCmd` after that point (the strand decision, the receiver, creel, the session start) returns through the `fail` closure, which writes a bare `output.Err`. The card does not change it, and no row has a failing launch after a reconcile that removed a pair, so a removed pair would go unreported exactly when the start fails. `lyx cleanup --apply` has the equivalent row (a segment failing after a moved branch sets `partial` true), which orch start lacks.

### [LOW] A failed board push prints two way-forward lines in the reconcile verb

**Location:** `_lyx/plan/09-push-retry-and-record.md:89`

**Issue:** `hubConfigFailure` and `warpBindingFailure` in `internal/fabriccli/fabric.go` print the failure's detail followed by `reconcileFailureWayForward`. After card 9 the detail of a board push failure is a `*PushFailedError` text that already ends in `way forward: ` and the class's line, and the card rewrites the appended text without the rule it gives `commitRecordsVerb` (append only when some failure is not a `*PushFailedError`), so the operator reads two lines for one failure. No existing fabriccli test asserts the text, and the card plans none for the reworded one.

## Rejected

- Prime flag and class rows: card 9's `TestPushFailureClasses` pushes a rejection and a 403 from the prime through `Fabric.PushBranch`, which is the path Finalize's parent push and Publish reach, and asserts `Prime` set and the prime form of the line for both; card 1's rows pin the prime lines for `transport`, `auth` and `not_fast_forward`; the flag-set path is therefore failed by a test.
- Two-sided printing: the rule in card 9 for `commitRecordsVerb` reads each member of a joined error through `Unwrap() []error`, and its rows cover a join of `gone` and `auth`, and of `gone` and an unclassified error, each side's line once; `Finalize.pushParent` makes a single push, so its `errors.As` cannot drop a second side.
- Dirty line for the hub's `_board` worktree: card 15 gives a runnable form (`git commit` or `git checkout -- <path>` in the `_board` worktree, since `lyx fabric commit` is refused there), and cards 16 and 20 pin it exactly.
- Wake and mutation record: card 23 logs each mutation line at Info and asserts it in `TestWatch`, and card 20's rows assert `mutations` and `partial` on the cleanup envelope; the gaps are F2 and F8.
- Where a records-side failed push is run: card 1 names `lyx fabric push` in the pair's code worktree, in the prime's worktree for the prime weft, `lyx board sync` from the prime for Bolt, and pins each in `TestFailedPushRecord`.
