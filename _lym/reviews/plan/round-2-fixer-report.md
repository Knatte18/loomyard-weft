---
dispositions:
  - id: F16
    disposition: fixed
---

# Round 2 fixer-report

## Changes

- F16, fixed as proposed, with one wording change.
  The premise checks out against the code:
  - `probeLiveLocalWeft` runs only when a local branch exists and origin is ahead of it (`remoteleftover.go:124-125`).
  - `targetWeftTip` fetches only when no local branch exists and origin has one (`checkout.go:204-215`).
  - `Fabric.PushWeft` has no production caller (`doc.go:1521`).
  - `readSideOwnerArchives` is reached only through `Topology.ReadOwnerArchives` (`ownerarchive.go:273`).
  - `00-overview.md`, coverage-of-namespace-sites: I dropped the claim that card 6's Checkout row reaches `targetWeftTip`, and the sentence saying no one-repository row reaches `Fabric.PushWeft`, `probeLiveLocalWeft` or `readSideOwnerArchives`. The decision now lists those sites, the fetch inside `targetWeftTip` and `ArchiveOwnerTips` among the sites card 7 reaches.
  - `07-pair-lifecycle-namespace.md`: added `Topology.Checkout`, `ArchiveOwnerTips`, `Topology.ReadOwnerArchives` and `Fabric.PushWeft` to Uses. Added four bullets to `TestOneRepositorySecondMachine_AdoptPullAndReplay`: live local branch, checkout, records push and owner archives.
  - The wording change: "reports a fast-forward" became a `KindRepoAdvanced` mutation naming origin's tip, because `Add` records the fast-forward as that mutation (`add.go:565-578`). The checkout bullet also gains a fetch of the code store, so that `git switch` of the code branch can resolve it.
- Self-check fixes:
  - The heading of card 7's `TestOneRepositorySecondMachine_AdoptPullAndReplay` now names checkout, records push and owner archives.
  - The live-local-branch bullet now runs on its own slug and names its setup: adopt, `Topology.Remove` with no remote step, a `git branch` recreating the records branch at the removed tip, then one more push by the first hub. This is the setup of `TestAdd_LiveLocalWeftAgainstOrigin` (`add_remoteleftover_integration_test.go:114-127`).
  - `TestOneRepositoryPair_RecordsLiveInTheNamespace` gains a third row. It archives an origin-only records branch through `ArchiveWeftTipForTest`, which reaches `resolveArchiveTip`, and asserts the tag at `refs/lym/tags/archive/<slug>/<tip>`. `ArchiveWeftTipForTest` is added to Uses, and the overview's list names `resolveArchiveTip`.
- Gate repair: the overview's `## PATTERN changes` section declared `PATTERN-single-repo-hub` as `changes`, while the decision record declares it as `adds`.
  The gate blocked on both sides of that mismatch.
  The bullet now reads `adds`, matching the decision record. Card 8 creates `pattern/PATTERN-single-repo-hub.md` as a new unit, which also makes `adds` the right verb.
  `lym loom validate-plan` now reports only the informational `create-new-unit` finding.

## Self-check

- fix-consistency, LOW: the heading of `TestOneRepositorySecondMachine_AdoptPullAndReplay` was stale. Fixed.
- fix-claims, MEDIUM, merged with the fix-proposal MEDIUM: as written, the live-local-branch bullet's `Topology.Add` ran against a pair already adopted, so it could not reach `probeLiveLocalWeft`. Fixed with its own slug and the remove-and-recreate setup, not with the `AdoptLocal` route the claims lens suggested. The remove-and-recreate setup is the one an existing test already proves.
- fix-proposal, MEDIUM: the same finding as the fix-claims MEDIUM above. Fixed.
- fix-affected-set, MEDIUM: no one-repository row reached `resolveArchiveTip`. Fixed with a new row in `TestOneRepositoryPair_RecordsLiveInTheNamespace` and an entry in the overview's list.
- fix-affected-set, unchecked list: the lens did not confirm whether the records-side paths of `originBranchHead` and `lsRemoteHeads` lie on card 7's push and reconcile rows. It raised no finding, so this needs no disposition.

## Disputed

None.

## Deferred

None.

## Rulings

None.
