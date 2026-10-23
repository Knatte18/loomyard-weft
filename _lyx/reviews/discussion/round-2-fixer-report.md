# Fixer report — discussion review, round 2

Every finding's premise held against the code; all twelve are fixed in `_lyx/discussion/decision-record.md`, with the support log's round entry and question ledger updated.

## Changes

- F1 (BLOCKING): D10 now states that no idle release frees the parent lock while the owner's run is at Finalize and running or a verify marker for the run is live, in both paths: the loop's wake hook (`internal/loomcli/looplock.go`'s `loopIdleRelease`/`idleReleaseOn`) and batten's `loopLockStep`. Scope's In list names both checks, and the lock criterion names the waiting fixer session and both paths.
- F2 (MEDIUM): D2 records that `MergeIn` resolves a branch name at merge time and that `pickMergeSourceSHA` merges a local SHA ahead of origin's. It decides that fabric's merge-in gains a given-SHA merge, which Merge-In passes the fetched upstream tip or the tip read under the lock. Scope's In list names `internal/fabricengine`'s merge-in. Fixed differently on one clause: the proposal said "Merge-In fetches through its own step so a fetch failure is Stuck". Merge-In already fetches through `ReadParentTips(true)` (`internal/landingshed/mergein.go:224`, `internal/fabricengine/parenttips.go:42`), and a failure there is an error stop that keeps its transient resume. D2 now keeps that error stop and takes the upstream tip from that read, never from the verb's best-effort fetch. A fetch failure is dropped from the "stay Stuck as today" list, which had named it wrongly.
- F3 (MEDIUM): D11 adds that the shed driver stencil's `binary_changed` re-launch rule excludes the landing halts and hands them back. Scope's In list names the stencil, and the re-step criterion covers the LLM driver.
- F4 (MEDIUM): D14 drops the `profiles.yaml` rewrite and keeps the file seed-only. Profiles and `run_settings` are now migrated at read time through `PrepareRunSettings`, which `checkProfiles` and `LayerRunSettings` both call. The reconcile criterion no longer lists `profiles.yaml`, a new criterion covers a profile holding `webster_max_bounces` and `merge_max_bounces`, and Scope's config line says "migrated at read time".
- F5 (MEDIUM): D10 step 4's kind cap is now "a test file of the hub's language". It names where the patterns are declared (beside `internal/planparser`'s `sourceExtensions`), gives the C# pattern, and has a language the table lacks treat every changed file as a test file. Scope's In list names the patterns, and the over-cap criterion has a C# row.
- F6 (MEDIUM): D13 restates the deploy bound: a run at perch budget 0, such as one under the `docs` profile, deploys code with no review, bounded only by the landing gate, D10's caps and `deploy_branches`.
- F7 (MEDIUM): D7 now builds PR-Review as a new `PRReview` engine in `internal/landingshed`, registered in `shedrecipe`'s map and reading its row's `pr_review` setting through `Env.Setting`. Its rationale is replaced, and it states that the row is no Command row.
- F8 (MEDIUM): the declared `PATTERN-recipe-owns-failures` text now allows Finalize's route back through the reroute edge.
- F9 (LOW): D11 defines the landing halts by producer, and its driver row, `arbiterstop` refusal, batten re-step skip and stencil exclusion all match them, so Webster's reroute-cap halt keeps today's handling. The hand-back criterion says so. Fixed differently on one point: the proposal's "Merge-In or Finalize" producer match for the reroute-cap prefix is narrowed to Finalize, because Merge-In has no reroute edge (D1) and can never write that prefix.
- F10 (LOW): D10 step 4 runs the tree-scoped command at a darn Finalize over a diff wholly under `tree_scope`, and D8's tree-scope clause names Finalize.
- F11 (LOW): D4 states `merge_fix_attempts` as fixer turns per row entry, the unit `finalize_fix_attempts` uses, and says an `escalate` verdict spends no attempt.
- F12 (LOW): D10 step 4's no-review halt keys on the Code-Review perch's effective budget: its own `code_max_bounces`, or `review_max_bounces` when that key is null.

## Self-check

- fix-claims: no findings.
- fix-proposal, MEDIUM, D7 misreads `PATTERN-command-row`, which covers side-action rows and forbids a new engine. Fixed with an own change: D7 now gives the reason the pattern does not apply, namely that the row is the seat of the future human PR review that will gate the landing, not a side action, and runs no command.
- fix-proposal, LOW, D14's read-time Warn repeats at every read and names no way forward. Fixed: D14 says the repetition lasts until the operator edits the key, and that each Warn names the module, the key and the way forward of renaming or removing it in `profiles.yaml` or the entry's `run_settings`.
- fix-consistency, MEDIUM, D2's new fetch-failure Stuck contradicts "as today". Merged with the fix-way-forward MEDIUM below and fixed there.
- fix-way-forward, MEDIUM, D2's fetch-failure Stuck has no way forward, and today's fetch failure is an error stop. Fixed: D2 keeps today's error stop, and the F2 entry above is marked fixed differently.
- fix-consistency, LOW, the tree-scope criterion and the `PATTERN-verified-tree` plan-writer note omit darn Finalize. Fixed: both now name it.
- fix-consistency, LOW, the declared `PATTERN-parent-lock` text lists only a large merge and an over-cap fix as reroutes. Fixed: it also names a loom conflict and a spent fixer cap.
- fix-consistency, LOW, D13's `parent_lock_idle_release_min` rewrite omits D10's Finalize exception. Fixed: the key's description gains it.
- fix-consistency, LOW, D4 states the resumed fixer run twice. Fixed: the duplicate clause after "an `escalate` verdict spends no attempt" is removed.

## Disputed

None.

## Deferred

None.
