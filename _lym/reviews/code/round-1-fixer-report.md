---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The anchored block is flagged as proposed, but the flag's way forward is not the proposed `--set` command. `setEntry` (internal/yamlengine/layers.go:399) replaces the value node and keeps comments only, so `--set` would drop the anchor and strand its aliases. The line asks for a hand edit in the editor instead."
  - id: F3
    disposition: fixed-differently
    reason: "Flagging any defaults mapping that holds a merge key would also flag a default whose merge key supplies only other keys, with an untrue line. The drop is flagged only when the mapping still holds jacquard once its own key is left out (`holdsBesideOwnKey`)."
  - id: F4
    disposition: fixed
---

## What changed

- F1: rows in `TestReinterpretJacquardBlocks` for a merge-supplied defaults entry on the set and drop paths, and for a jacquard block with no tapestry block, with a new `wantUnchanged` field.
  The flag-only return is covered by the drop-path row.
- F2: `ReinterpretJacquardBlocks` flags a differing jacquard block that holds a YAML anchor and leaves it, with a test row.
  Fixed differently: the proposed `--set` way forward was not used.
- F3: `reinterpretJacquardDefault` flags a default that a merge key would still supply after the drop, with rows for that case and for a merge key supplying other keys.
  Fixed differently: the proposed flag on every merge key was not used.
- F4: the `Default("jacquard")` assertion and its doc line are gone from `TestTemplateLadder`; the commit carries a `Loosened-Check` line.
- From the self-check: an anchored `defaults.jacquard` is flagged and not set or dropped, the anchor way forward is the one above, and `ReinterpretArrasDefault` flags a default a merge key also supplies, each with a test.

Commits: `1c8243c5d` (F1, F2, F3), `f9d4c7dac` (F4), `89c7dff67` (self-check fixes), and one after them for the comment repair.
`lym gate test` on `./internal/runprofile`, `./internal/hubreconcile` and `./internal/configreg` passed in tier0 with no failures after the self-check fixes.

A comment check then reported two comment lines wrapped mid-sentence, one in `reinterpret.go` and one in `reinterpret_test.go`.
Both are rewritten as one sentence per line, with no code change, and `./internal/runprofile` passed again in tier0.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, anchored `defaults.jacquard` replaced or removed: fixed with own change (flagged and left).
- fix-consistency, LOW, `ReinterpretArrasDefault` half-drops a default a merge key also supplies: fixed with own change (the same `holdsBesideOwnKey` check). The function predates this round's fixes.
- fix-affected-set, LOW, anchored `defaults.jacquard`: duplicate of the first fix-consistency finding, fixed.
- fix-proposal, MEDIUM, the anchor flag's `--set` way forward is refused: fixed as proposed (the line asks for the anchor and its aliases to be written out by hand). Checked against `setEntry`; the refusal itself was not run.

## Disputed

None.

## Deferred

None.

## Rulings

None.
