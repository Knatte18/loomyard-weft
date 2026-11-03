---
format: 5
approved: true
language: go
---

# Plan: batten-remote-order

A batten run must never fail pass after pass because its task worktree is gone, and a slug's remote work must stay in order: a teardown's archive tag never goes out ahead of the weft tip it archives, and no removal or deletion of a slug's former pair touches a later run's branches.
The plan builds the gitrepo primitives first, then fabric's `Add` (tracking-ref drop, adopt-only re-adoption, teardown lock), the teardown's archive order, leased weft deletes and the cleanup sweep's lock, then the run-scope reconcile option, batten's re-adopting resume and its failed stop, the removal follow-up of D3, and the tackler rule of D5.

## Card Index

1 — gitrepo-lease-and-retry — retry caller-named refusal reasons and return ErrLeaseRejected from a leased delete
2 — gitrepo-branch-push-and-tracking-drop — push a named branch and drop its tracking ref under the remote-refs lock
3 — add-drops-stale-tracking — Add drops a former pair's tracking ref and upstream for a branch origin lacks (D8)
4 — add-adopt-only — Add re-adopts a live pair under an adopt-only option and types its permanent refusals (D1, D2)
5 — teardown-lock-add-discard-push — Add, the task-branch discard and synchronous pair pushes take the teardown lock (D7)
6 — remove-archive-order — a teardown pushes the weft tip before its archive tag and archives a diverged origin tip (D4)
7 — leased-weft-delete — every weft delete on origin is leased on the archived tip and a moved branch is kept (D6)
8 — cleanup-teardown-lock — the cleanup sweeps try each slug's teardown lock and skip a held slug (D7, D6)
9 — reconcile-absent-pair — a run-scope reconcile may judge a run whose pair is absent (D1)
10 — batten-resume-readopt — a batten resume re-adopts a live pair whose task worktree is gone (D1)
11 — records-tag-reads — fabric reads a slug's records tags and a records branch's file (D2)
12 — readopt-failed-stop — a pair that cannot be re-adopted fails the run once with a way forward by row (D2)
13 — remove-follow-up — a removal of a batten run's pair names the step that settles its record (D3)
14 — tackler-remote-check — the tackler reads the remote tip before judging a commit absent (D5)

## Shared Decisions

### Decision: d5-no-prose-test

- **Decision:** Card 14 adds the tackler's `git ls-remote` rule as static stencil prose with no test, departing from the D5 acceptance criterion "the rendered tackler prompt carries the `git ls-remote` rule".
- **Rationale:** `PATTERN-test-speed` binds over the decision record: "No test pins a stencil's static prose"; the rule adds no marker and no Go-filled value, so review guards its wording.
- **Applies to:** card 14

### Decision: d3-sites-outside-fabric

- **Decision:** D3's follow-up is carried where the batten record can be read: `lym fabric remove`'s result warning (card 13, through `internal/pairteardown`, which already reads batten's record), the owned-elsewhere ender's refusal in `internal/tendercli`, batten's own resume refusal in `internal/battencli`'s `preflightRunOwner` and its cancel refusal in `judgeOrigin` when either quotes a fabric way forward naming `lym fabric remove`, and landingshed's stopped reason (`stoppedReason`, from Parent-Align and Finalize), which takes the follow-up as a value loomcli's `landingDeps` passes in. Finalize's adopter-first stop keeps its text, as D3 requires.
  Finalize's stop on the path where the parent already holds the task's work passes no follow-up either, since the work has landed while the record still reads before Land, so a cancel would release a landed entry.
  `internal/adoptcli/adopt.go` and `internal/cleanupcli/reconcile.go` refuse their verb with `sharedStopText`, which keeps only the branches every pair shares (Bolt and the prime's records branch); the prime carries no batten run record, so the follow-up there is always empty, and adopt's per-branch warning line is a report the verb goes on past.
  `internal/hibernateengine/wake.go`'s `runStep` words the wake report's step for each held run while the wake goes on, and names `lym cleanup --apply` after a gone pair's way forward; it is a report, not a stop or refusal, so it carries no follow-up.
  The way forwards worded inside `internal/fabricengine` (the failed-push record's gone-branch text in `FailedPushWayForward`, `lockPairTeardown`'s re-run, the destructive gate's local-only refusal) stay as they are; `lym fabric remove`'s own refusals that name re-running itself, and the Worktree-Create row's `incompletePairRemedy`, whose way forward is the resume of the same run that re-creates the pair, are not ways forward that leave a record behind.
- **Rationale:** `PATTERN-fabric-isolation`: fabric learns nothing of the board's or batten's records, so it cannot read a batten run record or name a batten verb, and a hook letting batten append to fabric's way forwards would be a seam shaped for one module, which the same entry forbids; this departs from D3's "every stop or refusal" for those fabric texts, and the `lym fabric remove` warning reaches the operator after any such removal.
- **Applies to:** card 13

### Decision: land-way-forward-in-battencli

- **Decision:** D2's Land restore is computed in `internal/battencli` over general fabric reads (card 11: `SlugRecordsTags`, `RecordsBranchFile`, `RecordsBranchHasUpstream`); the `lym fabric remove` warning at Land (card 13) names running `lym batten run <slug>`, whose failed stop then names the exact restore.
- **Rationale:** `PATTERN-fabric-git` keeps git on the records repository inside fabricengine and `PATTERN-fabric-isolation` keeps the seams general; `internal/pairteardown` is imported by battencli, so it cannot call battencli's builder, and the pass that meets the gone pair computes the restore from the tags the removal left.
- **Applies to:** cards 11, 12, 13

### Decision: not-live-typed-answer

- **Decision:** `Add`'s adopt-only not-live answer is `*fabricengine.ErrNotLive` naming the branch, matching the sentinel `ErrPairNotLive` under `errors.Is`, so batten tests the closed set by sentinel and names the restore of only the branch that is not live.
- **Rationale:** D2 needs both a sentinel test and the branch it concerns.
- **Applies to:** cards 4, 12

## PATTERN changes

- `PATTERN-slug-remote-order` — adds: Touching a slug's pair or its origin branches: under fabricengine's per-slug teardown lock, shared for each pair push outside a removal, exclusive for Add, Remove, task-branch discard and cleanup sweep; a weft branch delete on origin is leased on the origin tip read.
- `PATTERN-push-both-sides` — changes: Pushing run records from Go: code branch goes along; a push refused after retry leaves a failed-push record and creel notice. A teardown's push of a weft branch it archives and deletes goes alone and only logs a refusal.
