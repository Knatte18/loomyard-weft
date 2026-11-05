# Round 2 fixer-report

## Changes

- **F1** (fixed as proposed, commit `e0d2e01f`): `FollowTemplate` in `internal/yamlengine/follow.go` parses the base only when `haveBase` is true, so a corrupt base no longer fails a reconcile that does not read it.
  The parse error still wraps `ErrInvalidBase`.
  The doc comment on `FollowTemplate` and `docs/shared-libs/yamlengine.md` say `base` is not read while `haveBase` is false.
  `follow_test.go` gains a row with base `"- not\n- a mapping\n"`, `haveBase` false and no error, which fails against the old code.
  `lym gate test` is green for `./internal/yamlengine`, `./internal/configsync` and `./internal/hubreconcile`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings. It inferred that the existing corrupt-base rows in `configsync` and `hubreconcile` keep `haveBase` true; the gate run over both packages confirms they pass.
- fix-proposal: no findings.
- fix-affected-set: no findings. It noted that an unreadable base still fails an absent-file reconcile, which is a read failure outside F1's premise and is pinned by the `baseIsDir` row.

## Disputed

None.

## Deferred

None.
