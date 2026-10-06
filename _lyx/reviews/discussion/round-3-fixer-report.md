# Round 3 fixer report

All nine findings are fixed in `_lyx/discussion/decision-record.md`, and F6 in `_lyx/discussion/support-log.md`.

## Changed

- F1 (D7): the plain-merge way forward is removed.
  The refusal now names `git merge --strategy ours` for the run's own abandoned commits, and for commits that may be someone else's it says not to re-run the reset and leaves the rewind to the operator.
  The Bound states that a remote-only commit becomes reachable only through the operator-run `ours` merge, and Open risks names the residual.
- F2 (D7): the order inside the pair-warp reset is now stated (gate checks, then fetch and ancestry check, then remote update, then checkout rewrite), so the local refusal comes first.
  A fetch failure joins the network and authentication clause, and a repository with no remote configured joins the skip list.
- F3 (D4, D6): D4's batch-failed reason names the `run --fresh` steps when the record lists uncheckable paths.
  D6's `rebaseline --card` refuses a failed batch with uncheckable paths and names the same steps, and its Bound says so.
- F4 (D6): the Bound states that a dead batch's strand may still be working on the old card when the restamp lands, and that `recover-batch` ends the window by stopping the strand and archiving a late report.
- F5 (D10): the rationale and Bound now describe the dormant recorded-versus-launch-model comparison; they claim no drift check and no speed effect.
  The `Run` launch-time write of `AssertedModel` joins the removal list.
- F6: the support-log rejected-alternative line now says reset can tell the commits apart only while the local HEAD still holds them.
- F7 (acceptance criteria): rows added for the network or authentication failure, the refusal for someone else's commits, the no-remote, no-branch and `SkipPush` skips, the local refusal coming first, a dead batch with a live strand, the uncheckable cases for D4 and D6, and `run` recording no `assertedModel`.
- F8 (D1): the rationale states that the card-order guarantee holds for contiguous batches, as the identity batcher's are.
- F9 (D1): the finding is attributed to the consumer card, and the dispatch scoping is stated as dropping a finding by that attribution.

## Departure from the review's wording

F9's Fix said a pair with a begun card on either side is dropped.
`planglyph.ValidateDispatch` scopes a pure finding by its attributed card alone, so only the consumer's state can scope the pair.
D1 now reports a pair when its consumer card is unbegun, whatever the later producer's state: a later producer already begun means the run passed it out of card order, and refusing is right.

## Deferred

Nothing deferred.
