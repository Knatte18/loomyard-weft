---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: The proposed step order "start, each tier's start and end, end" describes the check's record steps in internal/adoptcli/check.go:147, not the adopt's steps. The doc now lists the adopt's real order (hold, pairs, takeover, catchup, deploy, workspace, check, beam) beside the restored envelope field list.
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

## Changes

- F1, `internal/adoptcli/doc.go`: restored the envelope field list under `# The envelope` from base 3d3b4812.
  Added the adopt's step order, the effect of a failing catch-up or deploy, and the `--no-orch` and check-worktree contracts.
- F2, `internal/yamlengine/doc.go`: cut the per-function catalogue to one overview sentence naming the per-file operations.
  Cut the Reconcile, FillMissing and MissingKeys open-map bullets and the Reconcile and Prune per-name rules their doc comments state.
  Kept the Reconcile refusal's way forward and why a dropped name still follows the template.
- F3, `internal/batcher/doc.go`: cut the Active restatement and the term-by-term estimator narration.
  Merged the Active load contract into one place.
  Kept the retired-key refusals, boundary rules and the recorded-partition invariant, and defined the start base once.
- F4, `_lym/landing/summary.md`: "162" is now "161" in the Size row and the "130 of the 161" sentence.
  This is a working-tree edit only; `_lym` is never committed by the fixer.

Commits: F1, F2 and F3 each have a first commit and a self-check follow-up commit.
`lym loom lint-comments --base 3d3b4812` passes.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings; F2 and F3 have no set premise and were left unchecked.
- fix-consistency, MEDIUM: F1 step sentence names the wrong steps. Fixed as proposed.
- fix-consistency, MEDIUM: F2 overview claims no operation replaces a file value, which is false for set operations. Fixed as proposed by dropping the sentence.
- fix-consistency, LOW: F2 Reconcile sentence says a template edit reaches "the file". Fixed as proposed, now "the loaded value".
- fix-consistency, LOW: F3 left "start base" undefined. Fixed as proposed, defined in one clause (the `StartBase` doc at estimate.go:77 carries the zero value).
- fix-consistency, NIT: summary.md edit is uncommitted. No change; expected, `_lym` is not committed by the fixer.
- fix-proposal: its findings duplicate the fix-consistency ones above, with the same dispositions; its NIT on summary.md matches the fix-consistency NIT.

## Disputed

None.

## Deferred

None.

## Rulings

None.
