# Fixer-report, merge round 1

## Changes

- F1 (`pattern/PATTERN-verified-tree.md:10`), fixed differently.
  The `round` gate's site list now names both verify-gate fixers, the fixer fork and the cold fixer `lyx webster fix-verify` spawns, as the reviewer asked.
  Not made: the proposed clause that each fixer runs the `round` call "through `Verify` with `BaseCommand` set to the told base command".
  `internal/websterengine/verifyfixer.go` only renders the call with `deps.Gate.GateCall("round")` into the prompt's `fix_gate` marker (`render.go:336`), and no non-test code in `internal/websterengine` calls `Verify` or sets `BaseCommand`, so the clause is false for the cold fixer and unsupported for the fork.
  The sentence now says the fixers share one prompt body and each run the hub's `round` call, rendered into `fix_gate`, before they report.
- `internal/websterengine/doc.go` needed no change: it already describes both fixers sharing `webster-body-verify-fix` and the `round` call (lines 521-526, 534).

Commits: a69913b9, plus a follow-up commit narrowing the sentence.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `PATTERN-verified-tree.md` lines 3 and 9 still name the retired Webster-Burler-Fix round gate. Disposition: deferred, see below.
- fix-proposal, MEDIUM: the `Verify`/`BaseCommand` clause does not hold for the cold fixer. Disposition: fixed with my own change, as above.
- fix-affected-set, LOW: same lines 3 and 9. Disposition: deferred, see below.
- fix-affected-set, unchecked item: the `Verify`/`BaseCommand` claim; resolved by the proposal-lens fix above.

## Disputed

None.

## Deferred

- Lines 3 and 9 of `pattern/PATTERN-verified-tree.md` naming Webster-Burler-Fix, found by two lenses.
  The review rejected them as a retired segment the task's own edit left, not part of this interaction.
  The same retired name also stands in `contracts/specs/loom-status-spec.md:115`, `internal/verifytree/verify.go:61`, `internal/webstercli/verify.go:1,30`, `internal/loomshed/doc.go:40` and `pattern/PATTERN-gate-self-check-parity.md:15,17`.
  Rewriting lines 3 and 9 also needs a decision on where the `round` call runs through `Verify` once that segment is gone, so it belongs to a sweep of the retired name, not to this finding.
