---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: ruling
    reason: "The parent ruled (decision record, Added after Discussion 2026-11-21) to declare a PATTERN-code-conventions change admitting lym-owned records-namespace branch names as named constants in fabric; carried out in the overview and the docs card, and the parent then declared the bullet in the record's PATTERN changes."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The guard sits in pushClassified for side bolt, before its upstream read and push, not in pushJob.push: a refusal from pushJob.push falls to job.fail (pushclassified.go:166), which writes a failed-push record that later pushes retry and appends a second way forward."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed-differently
    reason: "The owed-push row became a records-replay row: retryWeftPush is the rejection replay (pushanchored.go:207, coalesce.go:177), reached by a rejected Fabric.PushBranch, not by an owed-push re-run; it sits in the second-machine test, which has the two hubs a rejection needs."
  - id: F9
    disposition: fixed-differently
    reason: "Took the review's merge route but folded card 1's namespace type and refspec read and card 2 whole into card 3, keeping card 1's upstream reads as their own card (renamed gitrepo-refspec-upstream), and gave LocalRef its first caller in card 3's namesRef. The allowlist alternative was rejected: the allowlist admits only functions that tests of other packages share (cmd/lym/testonlycode_linux_test.go:41-46, pattern/PATTERN-test-only-code.md:28)."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed-differently
    reason: "The row's lock file makes git 2.53.0 reject with `reference already exists`, never `cannot lock ref` (probed under a lease on absence, a lease at a tip and a plain push), so card 4's classification now counts `reference already exists` as transient and the row expects that line."
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
---

# Round 1 fixer-report — plan

This session found every fix below already on disk in `_lym/plan`.
An earlier fixer session of this round made them and wrote its report as `round-1-fixer-report-20261121T135453Z.md`.
This session checked each finding against the plan, found its fix present, and changed no file.
`lym loom validate-plan` answers `ok: true`, with one informational finding (the new PATTERN background file).

## Changes

Card numbers below are the new ones; the review's numbers are in parentheses.

- **Restructure (F9, fixed differently).** Each new production function now lands with its first caller.
  - Card 6 (clone-single-repo-flag) was merged into card 4 (was 5): its targets, Intent, bound and test row moved there, then `lym loom remove-card 6`.
  - Card 2 (records-layout-read) and card 1's `RefNamespace`, `Branch`, `Tag`, `LocalRef`, `RemoteFetchRefspecs`, `TestRefNamespace` and `TestRemoteFetchRefspecs` were folded into card 2 (was 3), then `lym loom remove-card 2`.
  - `namesRef` now maps `<dst>` through `ns.LocalRef`, so `LocalRef` has a caller in the same card.
  - Card 1 keeps only the refspec-following upstream reads and is renamed `01-gitrepo-refspec-upstream.md`.
  - The remove-card calls rewrote every cross-reference to the new numbering.
    The overview's Card Index, the Applies-to lines and card 8's (was 10) card list were updated.
- **F1.** In card 4, `pushCloneSeed` is a destruction-gate executor.
  Before the records push and before each restore, it runs these checks in order:
  - containment through `checkPathRequest` on the hub path;
  - ownership, as `ownedFreshlyCreatedPath(seed.hubTok)` plus a check that the ref is the records branch or board branch named at the precondition check, leased at the tip that check observed;
  - dirtiness, as `dirtinessNA` with a stated reason;
  - force, false.

  A failed check refuses before any push.
  The card states that these checks are a fail-loud wiring guard.
- **F2 (ruling).** See Rulings.
- **F3.** Card 2's `TestDeleteRemoteBranch` rows use `refs/lym/heads/x-records` and `y/x-records`.
- **F4 (fixed differently).**
  - Card 2's `pushClassified` (added to Edit) refuses an upstream-less board in a one-repository records store.
    The refusal comes before the upstream read and the push, and names the same clone again with `--reset` as the way forward.
  - It writes no failed-push record and runs no replay.
  - Card 4 states the bound for the reconcile path.
  - Card 5 (was 7) gains the unseeded-board row, which also asserts that no failed-push record is written.
- **F5.** Once the board push lands, card 4's `pushCloneSeed` sets `branch.main.remote`/`branch.main.merge` (= `refs/lym/heads/main`).
  Card 5's `TestCloneSingleRepo_SeedsTheNamespace` asserts the merge key and that `HasUnpushed` answers false.
- **F6.** Card 4's sentence claiming no test reaches `AbandonSeededClone` was deleted.
  Card 5's failed-seed test gains the abandoned-seed row and the two-URL row.
- **F7.** Card 7's (was 9) one-repository never-pushed row first creates a same-named code branch under `refs/heads/` and asserts it survives with no `remote_branch_deleted` entry.
- **F8 (fixed differently for one row).**
  - The overview's coverage decision names the sites card 7 reaches.
    It adds checkout via card 6 and the unseeded-board refusal via card 5.
    It names `Fabric.PushWeft`, `probeLiveLocalWeft` and `readSideOwnerArchives` as reached by no one-repository row.
  - Card 7 gains three rows: a leftover-replacement row (`probeWeftLeftover`), a records-replay row (`retryWeftPush`) and a hand-over row (`TakeOverRun`).
  - The unsupported sentence "The hand-over's records-side reads are…" was dropped.
- **F10.** Card 4's join fetch goes through `gitrepo.New(dir).Fetch()` under the remote-refs lock.
- **F11.**
  - Card 4: under a non-identity namespace, `recordedAnchorOnWeft` skips its `HEAD`/`branch --show-current` fallback, and the subpath is checked against the warp tree.
  - Card 5's join test gains the `--reset` row: no `main-weft` and a missing `--subpath`.
- **F12 (fixed differently).**
  - Card 4's transient set is now `(failed)`, `cannot lock ref` and `reference already exists`.
    The card says why the last one is transient.
  - Card 5 gains the lock-file row, which expects `reference already exists`.
- **F13.** Card 3 (was 4) drops `hasWeftEvidence` and `RecordsBranchFile` from Edit.
  It states that `readSideOwnerArchives` fetches `ns.Tag(tag)`, that `RecordsBranchFile` needs no change, and that `hasWeftEvidence` is unchanged.
  The ImpactSummary now says "Fifteen".
- **F14.** Card 2 puts `TestRefNamespace` in a new tier0-only gitrepo test file and `TestRemoteFetchRefspecs` in `remote_integration_test.go`, the tier1 file of the `RemoteURL` tests.
- **F15.** Card 4's Seed bullet loses the clause about an origin whose `HEAD` names no branch.

## Self-check

This session made no change, so it spawned no fork.
The earlier session's fork pass checked the change set above; its lens reports are on disk as `round-1-fixer-report.lens-<lens>.md`, and its judgments stand:

- fix-consistency, LOW — card 7's Intent overclaimed coverage after the overview was narrowed: fixed.
  It now points at the overview decision.
- fix-consistency, LOW — card 8's Card Index line omitted the `PATTERN-code-conventions` work: fixed.
- fix-consistency, note (outside its lens) — whether a joined board has an upstream: no finding.
  The claims and proposal lenses confirmed `ensureBoardWorktree`'s `--track` form, and card 2 now names it.
- fix-proposal, MEDIUM — the F12 lock-file row yields `reference already exists`, not `cannot lock ref`: fixed differently, as F12 above.
- fix-claims, MEDIUM — the F4 refusal from `pushJob.push` would be recorded as a failed push with a second way forward: fixed differently.
  The guard moved into `pushClassified`, before the push, and returns unrecorded.
- fix-affected-set, MEDIUM — the F4 guard did not say whether the board replay (`retryBoltPush`) could still push: fixed, merged with the claims finding.
  No push runs, so no rejection triggers the replay, and the card says so.
- fix-affected-set, MEDIUM — the coverage decision implied every other site was reached, but `Fabric.PushWeft` and `targetWeftTip` are not reached by card 7: fixed.
  The decision now names checkout via card 6 and lists `Fabric.PushWeft` as unreached.
- fix-affected-set, LOW — `remoteTagsUnder`'s archive-coverage and leftover reads were unnamed: fixed.
  The decision names both under the remove and add flows.

## Disputed

None.

## Deferred

None.

## Rulings

- F2: the parent ruled that lym-owned branch names inside the records namespace (`main` under `refs/lym/heads/`, `lym-bolt`) are named constants defined once in fabric.
  It asked for the change in the overview's PATTERN changes, and for the docs card to write it into `PATTERN.md` and `pattern/PATTERN-code-conventions.md`.
  - Carried out: the overview bullet, card 8's Edit targets and its Intent text, and the constants' citations in cards 2 and 4.
  - The decision record's own `## PATTERN changes` bullet is outside the write surface; the earlier session told the parent once (creel #2195), and the parent declared it in the record (#2197, #2198).
    This session sent no message.
