# Webster refusals: short texts, right cause, real way forward

A batch refusal over an unresolved Create target or Rename pair was long, misleading, and ended in a dead end.
This change fixes all three.

- **Short text.**
  A `create-not-done` or `rename-not-done` detail no longer lists every searched file per target.
  It names how many files declare the unit.
  It lists at most three declarations of the same name elsewhere in the worktree, nearest unit first, and counts the rest as "and N more".
- **Right cause.**
  When the unit declares the member under the same name with only other parameter lists, the finding is a new blocking `signature-mismatch`.
  It leads with each declared signature and its file, instead of reporting the member as missing.
- **Real way forward.**
  `recover-batch` no longer names itself as the way past its own repeat.
  - A `create-not-done` or `rename-not-done` refusal offers amending the card's Create target or Rename new side, then `lym webster rebaseline --card NN`, then `recover-batch`, beside plain `recover-batch`.
  - A `signature-mismatch` refusal, or a create or rename refusal from a batch a recovery already tried, offers the amendment alone.
  - A batch whose counted recoveries are used up takes the reset-to-start or fresh-restart steps instead, since a failed batch records no amendment.
    This also applies to the existing delete-reference and `glyph-rejected` ways forward, and does not apply while an unrendered amendment is pending.
  - A record that recovery refuses as uncheckable still takes the fresh-restart steps.

Closes #61
Closes #62
Closes #63

## Design decisions

- `planglyph` owns the detail text and the `signature-mismatch` check.
  Its `donecheck.go` runs the check after the counterpart-package rule, so a member declared in the other package still counts as done.
- `signatureMismatches` matches on the same unit, owner chain and name, and fires only when the card's target names a parameter list.
- `websterengine` carries two new flags on the done-check error and picks the way forward in `createTargetWayForward`.
- `exhaustedRecoveryWayForward` is now shared by `recover-batch`'s `ErrRecoveryExhausted` refusal and the failed-batch way forwards, so the text stays in one place.
- No verb to retarget a create target was added; the existing card amendment plus `rebaseline` covers it.

## To check by hand

- Review finding `rename-not-done-flag-missing` (MEDIUM, scope) is left open in the `rename-not-done` handling of the new `createNotDone` flag in `internal/websterengine/recordbatch.go`.
  Check that a Rename pair that did not happen takes the create way forward.
- No regression check ran, and no run has exercised the new texts against a real refusal.
  Provoke a signature mismatch and a relocated create target with `lym webster record-batch` and `recover-batch`, and read the refusal text and way forward.
