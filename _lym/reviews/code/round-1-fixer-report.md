## Changes

- F1 (MEDIUM): `internal/hubforge/template_test.go` gained `failingBuilds` rows `Skip`, `Skipf` and `SkipNow`. Fixed as proposed.
  After the self-check, each row's `wantFirst` is anchored on the full phrase (`TB's Skip:`, `TB's Skipf:`, `TB's SkipNow:`) so a row cannot pass on another method's refusal.
- F2 (LOW): `failingBuilds` gained an `Errorf` row whose build records a non-fatal failure and returns. Fixed as proposed.
- F3 (LOW): the `Registration.Build` comment in `internal/hubforge/template.go` and the matching paragraph of `internal/hubforge/doc.go` now state that a build reaches no code whose result a once-only registration changes, such as a Bolt replay. Fixed as proposed.
- F4 (NIT): the `requireIntact` message in `internal/hubforge/template.go` now reads "take CopyHub or CopyRegistered in the test that writes". Fixed as proposed.

Gate: `lym gate test --tier 1 ./internal/hubforge` passed (47 of 47).

Commits: F1 and F2 landed in one commit and F3 and F4 in another, since each pair shares a file; the self-check's anchoring change to F1 is a third.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, the `wantFirst` of the `Skip` row is a substring of the `Skipf` and `SkipNow` refusals: fixed as proposed by the fork, by anchoring the three skip rows on `TB's <method>:`.

## Disputed

None.

## Deferred

None.
