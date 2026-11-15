# Fixer-report, round 1

Commit `0813d91f` fixes all four findings. The F1–F4 edits overlap on the same fragment lines, so they went in one commit instead of one per finding.
`lym gate test ./internal/darnengine ./internal/loomcli ./contracts/stencils` passed (tier0, 0 failed).

## Changes

- F1 (fixed as proposed): `darn-fragment-make-change-gated.md` no longer names a package, `-- -run` or any test flag.
  Item 3 is back to "3. Run no `lym gate test` yourself."
  The writer asks its parent through `lym creel post --parent --expect-reply` to run the reported test once through `lym gate test`.
  The header comment in the fragment, `contracts/stencils/stencils.go`, `internal/darnengine/spec.go` (both comments) and `internal/darnengine/doc.go` say the writer runs no gate and asks its parent for the single run.
  `_lym/landing/summary.md` records the parent route as the design decision, with `PATTERN-test-speed`'s no-scope-form rule as the reason.
  The summary file is edited but not committed, since `_lym` is not mine to commit.
- F2 (fixed as proposed): the request names the test exactly and its tier.
  The writer reads a pass as "no longer reproduces" only when the report shows the named test ran.
  A skipped test or one that matched no test sends the writer on to diagnose as usual.
- F3 (fixed as proposed): `gatedReproduceRun` is deleted from `internal/darnengine/spec_test.go` with both uses.
  `gatedNoGateStep` in that file and in `internal/loomcli/wiring_darn_test.go` is the restored item-3 line.
  The gated row's name is restored.
- F4 (fixed as proposed): the reproduce instruction opens item 1, ahead of "Edit the repository to make the change.", with no renumbering.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings (one note, not a finding: whether the parent reports that the named test ran rests on the parent's judgment, which the F2 design implies).
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
