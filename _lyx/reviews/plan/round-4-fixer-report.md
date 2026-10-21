# Fixer report — plan, round 4

Every finding's premise held against the cards and the code; all nine are fixed.
`lyx loom validate-plan` passes after the fixes and after the self-check's changes.

## Changes

- **F1** (fixed as proposed): card 1's `rewoundWayForward` second choice is now `git push origin <branch>` in the task worktree, a plain fast-forward that restores the dropped commits and writes the push entry ending the reflog test; the `ours` merge is gone.
  Card 9's Intent says `lyx fabric push` stays refused until the rewind is settled and why the `ours` merge is not offered.
  `TestPushFailureClasses` gains the two settle rows (`git reset --keep`, raw `git push`), each asserting the next `lyx fabric push` lands and removes the `rewound` record.
  Fixed differently in one part: the rows assert that `classifyRewrite` over `trackingReflog` no longer reads the branch as rewound. The proposed "the next reconcile no longer reads it rewound" was not used, because `ReconcileOrigin` is created on card 15, after card 9 (self-check C1).
  That half moved to card 15's rewound row in `TestReconcileOrigin_Judge`, which now reads the branch level after each settle.
- **F2** (fixed as proposed): card 4 creates `KindTagCreated` (`tag_created`), edits `internal/fabricengine_test#manifestObservableKind` (classified false), and gains an ImpactSummary.
  Card 4 says `setAsideTag` appends the entry only after the tag exists, with Target the repo root and Detail the tag name.
  Card 13's `SnapshotUncommitted` appends its tag the same way and lists the kind in its Uses.
  Card 16's rows assert one `repo_advanced` entry per fast-forward, an empty record for the judge-only scope, and one `tag_created` entry per set-aside tag of a Bolt replay.
- **F3** (fixed as proposed, extended by self-check P1): card 15's dirty line gives each path kind its directory: a pair weft in the pair's task worktree, the prime weft in the prime worktree, a code path in its task or prime worktree, Bolt in `_board`.
  Any weft path can also be committed or restored with `git` in the weft worktree; this covers a path outside the record pathspec that `lyx fabric commit` does not stage.
  The rewritten line names `lyx fabric pull` in the pair's task worktree, or in the prime worktree for the prime's warp branch.
  Card 16 pins the three dirty lines exactly; card 15's rewritten row pins the pair and prime lines exactly.
- **F4** (fixed as proposed): card 15's `WayForward` for dirty, gone and busy is the repair step alone.
  A new paragraph says each hop appends its own re-entry: `Summary` and card 20's refusal append `lyx cleanup --apply` from the prime, card 21 `lyx batten run <slug>`, card 22 `lyx loom resume`, and card 23 per case.
  `Summary` is now for the orch start note only, since card 23 builds its notice from `WakeRun`.
  Cards 15, 16 and 20 pin the split in their rows; cards 21 and 22 through F5's rows.
- **F5** (fixed as proposed): card 19's row uses a sentinel `WayForward` and asserts `Summary()` verbatim in the prompt line.
  Card 21's and card 22's stopped rows assert the fake's `WayForward` verbatim followed by the hop's re-entry.
- **F6** (fixed as proposed): card 1's first-failure row asserts the notifier's received line exactly; card 2's row asserts the mail's text equals it.
- **F7** (fixed as proposed): card 9 says Publish keeps `pushRejectedReason` for a `not_fast_forward` `*PushFailedError` and appends nothing to it.
  `TestPublish_StuckBeforePullRequest` gains a `transport` row and a `not_fast_forward` row.
- **F8** (fixed as proposed): card 19's `fail` closure carries the report's `mutations` and `partial` once the reconcile has run, and two new rows cover a failing launch.
- **F9** (fixed as proposed): card 9 gives `hubConfigFailure` and `warpBindingFailure` a `cause error` parameter; they append the reconcile line only when it is not a `*PushFailedError`.
  It edits `runReconcile` (validate-plan's caller-uncovered check) and adds a transport row plus exact-once assertions to `TestRunCLI_ReconcileBoardFailureFailsTheVerb`.

## Self-check

- fix-consistency C1, MEDIUM: card 9's settle rows used the reconcile, which card 15 creates later. Fixed differently: card 9 asserts through `classifyRewrite`, and card 15's rewound row reads the branch level after each settle.
- fix-consistency C2, LOW: card 13's Uses lacked `KindTagCreated`. Fixed as proposed.
- fix-consistency C3, LOW: card 15's hop list missed card 23's D3 stop (`lyx resume`) and the parked-push busy pair. Fixed as proposed: the busy pair now appends nothing, so `lyx resume <slug>` is not printed twice.
- fix-proposal P1, MEDIUM: the weft dirty forms gave no restore and no step for a path outside the record pathspec. Fixed as proposed: `git commit` or `git checkout -- <path>` in the weft worktree.
- fix-proposal P2, LOW: after a gone stop the wake notice said `lyx resume <slug>`, which only reaches an ended Halt. Fixed: card 23 appends `lyx cleanup --apply` for a pair stopped as gone. `WakeRun` gains a gone flag, `TestWatch` a row, and `TestWakeReconcileFromReport` a row.
- fix-claims: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
