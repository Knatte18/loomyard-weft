# Round 1 fixer-report

## Changes

- F1 fixed as proposed, by the restore variant: `tagSetAsideOrRestore` (`internal/fabricengine/destroy.go`, beside the other restoring reset, which the destructive-bypass scan requires) tags the set-aside commits and puts the branch back on the starting tip when a tag fails.
  `replayWeft` and `replayBolt` both use it, and `replayBolt` tags before it appends its mutation entries.
- F2 fixed as proposed, by the first variant: a push failure matching no named class is recorded under the new class `other` (`PushFailureOther`), so every failed push returns a `*PushFailedError`.
  A row in `TestPushFailureClasses` drives a non-repository origin URL and reads the record and the mail.
  `doc.go` names the class.
- F3 fixed as proposed, by the closure variant: `runReconciler` narrows the report through `reportOfRun` to the run's own entry.
  `TestReportOfRunKeepsOnlyTheRunsOwnEntry` pins the helper.
- F4 fixed with an own change: the first repair reverted `PATTERN.md` and `pattern/` to main's text plus the two declared entries `PATTERN-origin-reconcile` and `PATTERN-push-failure-visible`, but that left the overview past its headroom line, so the gate ran no test.
  The gate repair cuts the overview back under the line by shortening entries whose dropped wording already stands in their background files: durable-vs-ephemeral-state, hub-wide-config, hub-suffix, config-layers, push-both-sides, hub-store-gc, friction-capture, edit-directive, stencil-ownership, batcher-registry, testkit, model-choice-mapping and creel-store.
  The recipe-setting-lookup entry is shortened with its detail in a restored `pattern/PATTERN-recipe-setting-lookup.md`.
  Proposed change not made: leaving every other entry at main's text, which the headroom line does not allow once the two new entries stand.
  No background file other than the recipe one changed, so no wording is lost.
- F6 fixed with an own change: `cherryPickOnto` resolves `onto` before the reset, returns that error unchanged, and wraps every later `ResetKeep` failure in `ErrReplayDirty`.
  Proposed change not made: wrapping only when git refused over a local change, which needs git's English stderr (a self-check finding).
- F7 fixed as proposed: a retry that stops before it pushes is joined into the cause, so `FailedPush.Error` names it.
- F8 fixed with the doc variant: `Pull`'s doc comment states that a weft infrastructure failure returns before the warp arm.

## Disputed

- F5, every fix worse.
  The premise holds, but the proposed change does not resolve it: `cherryPickOnto`'s `git reset --keep` from the starting tip to upstream drops each set-aside commit, so git refuses over a dirty file under it anyway, with its raw text and no paths.
  Keeping the paths in `touched` makes the gate name them, so the first version of the change was reverted, and `replayWeft` carries a comment saying why.

## Deferred

- The self-check's LOW finding that `TestReportOfRunKeepsOnlyTheRunsOwnEntry` does not pin the call in the `runReconciler` closure.
  The closure runs a real origin reconcile over a hub, and no existing loomcli test drives it; a test for the one call would need a full hub and origin fixture, beyond this round.

## Self-check

- fix-proposal, MEDIUM, F5 narrowing does not stop the reset's refusal: disputed as above, change reverted.
- fix-proposal, LOW, F6 depends on git's English stderr: fixed with an own change, resolving `onto` first.
- fix-code-correctness, LOW, F6 markers: same finding, same disposition.
- fix-proposal, LOW, F7's way forward still reads reset or merge: not changed; the record's `Error` names the cause, which is what F7 asked for, and the class is still the push's own.
- fix-consistency, LOW, `replayBolt` also adds set-aside and seed paths to `touched`: no change, since the F5 change was reverted and the two functions agree again.
- fix-consistency, LOW, `tagSetAsideOrRestore` doc overstated its gate: fixed, reworded.
- fix-code-correctness, LOW, `reportOfRun` wiring unpinned: deferred as above.
- fix-affected-set: no findings.
