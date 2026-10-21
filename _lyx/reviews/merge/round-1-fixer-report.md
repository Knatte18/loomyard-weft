# Fixer-report, merge round 1

## Changes

- F1 (fixed as proposed), commit `12f7d4f8`: `PATTERN.md` carries the parent's overview text again for `PATTERN-hub-wide-config`, `PATTERN-hub-store-gc` and `PATTERN-friction-capture`.
  The restored bytes were cut from `PATTERN-transient-stop` and `PATTERN-driver-choice-single-site`, entries neither side changed; their background files already hold the cut detail.
- F2 (fixed as proposed), commits `99523663` and `46b33d1d`: the `PATTERN-origin-reconcile` entry ends "the sole judge of local branches against origin." as declared.
  The overage was cut from `PATTERN-pane-binary-resolution`; the overview is 15741 bytes, under the 15743-byte base, so the growth check passes.
- F3 (fixed as proposed), commit `13986727`: `TestRemoveLocal` and `TestSnapshotUncommitted` in `internal/fabricengine/remove_localmode_integration_test.go` write a file under the pair's linked root `.scratch`.
  The non-forced `RemoveLocal` succeeds only if the dirty gate ignores the link and its content, and its record is asserted to hold one `link_removed` at the root `.scratch` ahead of the warp `worktree_removed`.
  The snapshot test's exact path assertions now cover the populated link.
  `lyx gate test --tags integration ./internal/fabricengine -- -run 'TestRemoveLocal|TestSnapshotUncommitted'` is green.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW, the `PATTERN-pane-binary-resolution` cut dropped the detached window from its scope: fixed as proposed in `46b33d1d` ("strand pane or window").
- fix-affected-set, MEDIUM, `PATTERN-recipe-setting-lookup` is a parent-reworded entry the fix left at the task's short text: disputed, `premise false`, see below.

## Disputed

- `premise false`, fork finding on `PATTERN-recipe-setting-lookup`.
  The parent's change to the entry (`34261a22` to `860c5d03`) only dropped the words "executable-key" and "resolved"; it added no fact the task's text lacks.
  The three-way split in the overview was already present at the base, so its loss comes from the task's own cut to "looked up by key, never a field of its own on `Env`" at `ac1014b3^1`, with the detail kept in `pattern/PATTERN-recipe-setting-lookup.md`.
  That is the class the review rejects as the task's range alone (the Webster review judges it), unlike the three entries whose parent additions F1 restored.
  Restoring it would also need about 190 bytes cut from further unchanged entries.

## Deferred

None.
