---
verdict: BLOCKING
findings:
  - id: F8
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:118
    summary: "D5's bound and rationale assume the walk moves every file before any command loads it, but hubreconcile.Ensure runs only from start verbs (loom start and resume, batten arm, beam start, adopt, preflight, clone); after the build that ships this change, every in-flight run's next non-start verb strictly loads a hub base loom.yaml, a tracked worktree file or a Seed-Child-written local file that still holds flat keys, and D4 refuses it, so every running task on the hub halts until someone runs a start verb or the reconcile, against the Goal's carry-over with no operator step."
    fix: "In D5, replace the rationale's premise that the reconcile visits every file layer \"before any command runs\" with the true one: the walk runs at the first start verb on a new build. Then close the window with a Decision line, choosing one of these and naming it in D5: (a) every lym verb that loads the loom or tapestry config runs the stamp-gated hubreconcile.EnsureOrWarn before that load, as the start verbs do; or (b) while the build stamp is stale, the loader reads an old key at its new path in memory and warns, naming the walk, and D4's load refusal applies only once the stamp names the running build. Extend D5's second Bound line to cover files the walk has not yet reached on a new build, not only files that gain an old key after it. Add an acceptance criterion: after a rebuild that leaves the stamp stale, a non-start loom verb of an in-flight run whose hub base and local loom.yaml hold code_max_bounces: 2 resolves loom.code.max_bounces to 2 and does not halt."
prior:
  - id: F1
    status: resolved
    answer: "D4 now names one way forward per site (lines 89-95). Each reaches a fix at its site: lym config reconcile --apply runs ReconcileAll over the worktree and local files, --layer default runs ReconcileHubWideAt, and both move under D5. PrepareRunSettings names only the old and new names, so Seed-Child's existing wrapper prints the one way forward. The D4 criterion at line 209 covers every site."
  - id: F2
    status: resolved
    answer: "D6 now moves the file and its template base before FollowTemplate (lines 125-128), so an operator's changed value differs from the moved base and followUnit keeps it. The line-219 criterion pins the solid.tapestry case."
  - id: F3
    status: resolved
    answer: "D7 now marks the walk skipped on a failed board read or write, so ensure leaves the stamp stale and the next start verb's walk retries (line 142). It runs no carry when the hub-wide write was skipped (line 143), and the line-221 criterion pins the retry."
  - id: F4
    status: resolved
    answer: "Settled by the added entry \"F4: dotted names are rendered, input forms stay\"."
  - id: F5
    status: resolved
    answer: "D5 line 109 moves the prime copy's old keys before sparseSeed prunes it, and the line-215 criterion pins it."
  - id: F6
    status: resolved
    answer: "The mixed-build Open risk at lines 196-198 now separates the scalar-area refusal from the warn-and-default case."
  - id: F7
    status: resolved
    answer: "The D6 criterion at line 218 now compares the parsed YAML value."
---

### [BLOCKING] The walk does not reach in-flight runs' files before their next load

**Location:** `_lym/discussion/decision-record.md:118` (D5 bound), with the rationale at line 120 and D4's load refusal at lines 87-98.

**Issue:** D5's rationale says the reconcile "already visits every file layer, on every machine, before any command runs".
That premise is false.
`hubreconcile.Ensure` and `EnsureOrWarn` are called only by start verbs: `loomcli/arm.go` for loom start and resume, `battencli/arm.go`, `beamcli/cli.go`, `preflightshed/preflight.go`, `adoptcli/check.go`, `fabriccli/clone.go`, and the pair call in `battencli/wire.go`.
`ensure` in `internal/hubreconcile/walk.go:39-86` is the only path that runs the walk, and it is stamp-gated.

So between the rebuild that ships this change and the first start verb, nothing has moved the hub base `loom.yaml`, the tracked worktree files, or the local files Seed-Child wrote from `run_settings` and profile blocks.
Any non-start verb an in-flight run calls on the new build loads those files strictly through `loomengine`.
D4 then refuses the old names:

- in the hub base file, where `warnUnknownKeys` (`internal/configengine/config.go:302-329`) only warns today;
- in the local file, which `ValidateLocal` refuses today anyway, now with D4's message.

On a hub with several running tasks, which this board has, every one of them halts at its next config load until the operator or a start verb runs the walk.
The way forward D4 names does work, but needing it at all breaks the Goal's "carry over with no operator step".

D5's second Bound line was rewritten this round. It names only a file that gains an old key after the walk, not a file the walk has not reached yet on the new build.
The bound-breaker lens rated this MEDIUM.
I raise it to BLOCKING because it fires on the hub at this change's own landing, on every running task at once, not under an unusual interleaving.

## Rejected

- bound-breaker 2 (two ways forward if the old-name check lands in `ValidateLocal`): D4 line 91 already requires `PrepareRunSettings`'s refusal to name the old and new full names alone, and the line-209 criterion pins "Seed-Child's refusal prints one way forward". Where the check sits is plan detail that this requirement already constrains.

## Not raised: valid, but this incremental round admits only BLOCKING new findings

The prior-rounds rule allows a new finding only at BLOCKING.
The items below hold up on the evidence, but none reaches BLOCKING, so they are recorded here and not as findings:

- bound-breaker 3 and the existing-paths note (LOW to MEDIUM, D4 line 90): the hub-base way forward reaches a task session that `hubWriteRefusal` (`internal/configcli/hubwide.go:51-62`) refuses. Its text matches the existing `warnUnknownKeys` way forward, which also names the prime. F8's fix (a) or (b) makes it rare.
- bound-breaker 4 (MEDIUM, D5 line 114): under mixed builds, an older build's `--layer default --set code_max_bounces=3` after the walk loses to the moved new path's value on the next reconcile. The drop is reported to the orch, and the mixed-build Open risk does not name this loss.
- interactions F-int-1 (MEDIUM, D6 line 125): a profiles local file is read by `runprofile.Load` through `StackLayers`, but `reconcileSparseFile` leaves a seed-only module's file untouched, so D6 never moves it. Seed-Child's way forward then names the hub's `profiles.yaml`.
- interactions F-int-2 (MEDIUM, D6 lines 129-130): D6 moves "through the D7 helper", which D7 defines over a module-keyed settings object, yet D6 promises to keep the YAML file's comments.
- interactions F-int-3 (LOW, D5 line 116): `reconcileWorktree` only logs its results, so the record leaves unsaid where the walk gathers the worktree and local move lines into its one mail.
- walkthrough W1 (LOW, D6 line 129): no criterion pins a profile block that holds both an old key and its new path.
- handler: the Open risks line at decision-record line 195 and question-ledger Q10 still put the one-token question to the parent. The added entry "F4: dotted names are rendered, input forms stay" settles it, and the rubric would have the line dropped as a `consistency` fix.
- handler, focus 1: the `lym board upsert` refusal of an old-name `run_settings` key (`internal/boardcli/runsettings.go:144`) is not among the D4 criterion's sites, so no criterion reaches it. That package is outside `PATTERN-refusal-way-forward`'s bound modules.
