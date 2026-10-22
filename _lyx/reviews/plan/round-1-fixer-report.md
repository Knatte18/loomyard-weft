# Fixer report — plan, round 1

The F9 splits and the self-check's further splits renumbered the cards.
This report names cards by their new numbers; the review's cards map as 1→1, 2→2, 3→4, 4→5, 5→6, 6→7, 7→8, 8→9, 9→10, 10→12, 11→14, 12→16, 13→17, 14→18, 15→20, 16→24, 17→25.
The new cards are 3, 11, 13, 15, 19, 21, 22 and 23.
`lyx loom validate-plan` passes.

## Changes

- F1 (BLOCKING), fixed as proposed.
  Card 6 drops the `retryWeftPush` Edit target, `InstallOwnerReader`, `TestPushAnchored_OwnedElsewhereWeftIsNotReplayed`, the push-guard and `Fabric.Pull` Intent sentences, the doc.go mention and the ImpactSummary clause.
  Card 7 drops the `cmd/lyx#newRoot` target, the `InstallOwnerReader` Use, the pre-run install sentence and its ImpactSummary clause.
  The overview's push-retry Shared Decisions bullet is deleted, and its `Pattern-Departure:` continuation line now sits under the PATTERN-owner-machine bullet.
- F2 (BLOCKING), fixed as proposed.
  Card 24's Edit group gains `pattern/PATTERN-cli-cobra.md`, and its Intent adds the `adoptcli` deviation bullet.
  The self-check reworded the bullet to list packages, as its sibling bullets do.
- F3 (BLOCKING), fixed differently.
  The review proposed a `RetryDelay`, or a package-level seam, that defaults to one second and that the test sets to zero.
  I made neither change.
  A package-level seam is barred by `pattern/PATTERN-test-speed.md:86` ("never a package-level `var`").
  With a zero value that reads as one second, a test cannot set it to zero.
  Instead, card 4's `OwnerRequest` gains `Sleep func(time.Duration)` for the retry's wait, with nil reading `time.Sleep`.
  `TestSetOwners`' push-lock row passes a no-op `Sleep`, so no row waits a real second.
- F4 (MEDIUM), fixed as proposed, with one deviation.
  Card 12 keeps the owner trigger's state on `innerRunProducer`: its own `ownerSleep` watch, `ownerDue` and `ownerCovered`.
  `check` folds the clocks in before `checkChild`, and runs `ownerStep` after `checkChild` but before returning any end, over the status `checkChild` just read.
  `childWait`'s existing sleep watch stays where it is, because it feeds the notices' idle clocks (`notice.go:65,70`).
  The owner trigger gets a separate watch rather than taking that one over, so the notices' timing stays per-`Call`.
  The `childWait` Edit target is dropped.
  `TestInnerRun_OwnerCheck` gains the row for a sleep and a state change arriving on the same check.
- F5 (MEDIUM), fixed as proposed.
  In card 9, the resume's prime claim now runs only on a reachable reconcile.
  A failed claim warns and goes on to the owner verdict.
  With origin unreachable the resume makes no owner write and keeps D3's Warn and resume.
  `TestRunCmd_ResumeDispositions` gains both proposed rows.
- F6 (MEDIUM), fixed as proposed.
  Card 25's entry line is replaced with the review's line: 353 characters, under the 360 of `pattern.MaxEntryLineChars`.
- F7 (MEDIUM), fixed as proposed.
  Card 1 gains an Edit group with `pattern/PATTERN-leaf-packages.md` and `docs/overview.md`, an Intent sentence for the leaf section and the module-tree entry, and an ImpactSummary.
- F8 (LOW), fixed as proposed.
  Card 10 gains a Bound saying that, with origin answering, a run with no batten record reads owned here.
  Its `TestRunOwnerVerdict` now seeds a batten status file.
  Card 16 gains the same Bound for the run's owner.
- F9 (LOW), fixed as proposed.
  `lyx loom insert-card` created four cards:
  - card 3 for `ParentSlug` and `TestParentSlug`;
  - card 13 for `PauseOwnedElsewhere`;
  - card 15 for `LocalOnlyCommits` and `TestLocalOnlyCommits`;
  - card 21 for `DropLocal`, `DropLocalResult`, the destroy.go header, the guard-table rows and `TestDropLocal`.

  The consumer cards keep these symbols only under Uses.
  Every "card N" prose mention the tool reported was renumbered by hand.
  Card 13 does not list `tendercli#HubDeps` under Uses, since card 18 edits it later and the validator refuses that order.
  `HubDeps` is used unchanged, as before the split.
- F10 (LOW), fixed as proposed (the review's second option, after the self-check).
  Card 24's Step 2 keeps fabric's own message and way forward for every refusal except `*fabricengine.ErrBranchExists`.
  For that one, adopt names tagging the leftover tip under `archive/<slug>/<tip>`, then `git branch -D` and a re-run, and never `lyx fabric checkout`.
  `TestAdopt` gains that row.
- F11 (LOW), fixed as proposed.
  Card 14 drops the no-owner `TestChildSpawnError` row and the "another machine" wording.

## Self-check

- fix-consistency, MEDIUM, merged with fix-proposal MEDIUM: after F1's drop, no card stated that the weft push retry still replays. Fixed with my own change: card 6 gains a Bound naming the `retryWeftPush` replay and why it stays unguarded. The decision record's contrary premise is deferred (below).
- fix-consistency, LOW: `lens-review.md`'s guard-reach entry still described the dropped guard. Fixed: the entry gains a note that F1 superseded it.
- fix-claims, MEDIUM, merged with fix-proposal MEDIUM: fabric's own way forward for a leftover branch is `git branch -D` with no archive, plus `lyx fabric checkout` (`internal/fabricengine/add.go:65`). Fixed as proposed: the archive reword described under F10 above.
- fix-claims, LOW: on `checkChild`'s failed-stat and vanished-file ends, no fresh status is read. Fixed: card 12 runs no owner step on those ends and leaves `ownerDue` for the next `Call`.
- fix-proposal, MEDIUM ×2 (cards 10 and 16): the F8 Bound documents a run with no batten record reading owned here while origin answers, which is weaker than the offline path. Deferred (below).
- fix-proposal, LOW: the F2 bullet named functions instead of packages and left some out. Fixed: the bullet lists `fabricengine`, `pairteardown`, `boardengine`, `tenderengine`, `tendercli`, `battencli` and `landingshed`.
- fix-affected-set, LOW: adopt's owned_elsewhere stop misses a slug with no batten record. Fixed: card 24 gains a Bound stating it.
- fix-affected-set, LOW: more cross-package public symbols were created inside consumer cards. Fixed with four more cards:
  - card 11 for `ParentHoldsTaskWork`;
  - card 19 for `Engine.ClientAttached` and `TestClientAttached`;
  - card 22 for `ResumeInWindow`;
  - card 23 for `CloneSecond`.

  Their consumers' Uses, Intent, ImpactSummary and prose card numbers were updated.

## Disputed

None.

## Deferred

- The decision record's Open risks (`decision-record.md:250`, "those pushes are refused and recorded as failed pushes") and D4 (`decision-record.md:85`) both rest on a premise F1 showed is false: `retryWeftPush` replays.
  Card 6 states the replay as a Bound.
  Two changes need the parent:
  - correcting the record, which is outside this round's write surface;
  - closing the hole, which needs an Added-after-Discussion entry licensing an origin-Bolt owner read on the push path.
- The F8 Bound (cards 10 and 16): with origin answering, a loom or darn run with no batten record reads owned here at loom start, resume and Finalize, while the offline path reads the local Bolt and can refuse.
  Closing it means an owner read outside `ReconcileOrigin` for a reachable origin.
  The Constraints (`decision-record.md:223`) list every allowed non-reconcile owner read, and this one is not among them.
  The alternative is widening the reconcile's judged set beyond recorded pairs, which is a D4 change.
  Either is a decision for the parent, so the plan states the bound instead.
