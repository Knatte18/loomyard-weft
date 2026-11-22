```yaml
lens: fix-code-correctness
outcome: reported
```

Lens: fix-code-correctness, covering `RetireHardenLiveCopies`, the `FocusPath` rename and `RoundFacts`.

**Findings**

- **MEDIUM — `internal/gateslot/reinterpret.go`, the flagged line's way forward.** The line tells the operator to "remove gates.harden-live-copies with lym config reconcile", and reconcile cannot do that.
  - `gates` is a per-name map (`ConfigPerNameMaps`), and `ConfigOpenMaps` is only `tiers`, `tier_parallel` and `build`, so reconcile keeps an operator-named gate entry.
  - `applyReinterpretations` (`internal/hubreconcile/reinterpret.go`) writes the marker whether the rewrite changed the file or only flagged it. The rewrite never runs a second time, so nothing removes the entry.
  - The rubric counts a step that cannot be carried out as MEDIUM. The review prescribed this wording, so the fixer should decide whether to fix it differently. A removal step that works is `lym config gate` from the prime, or telling the operator to delete the line by hand.

- No finding for the other branches. I ran these inputs through the code:
  - Empty or non-mapping document: returns the input unchanged.
  - No `gates` map, or no `harden-live-copies` key: returns the input unchanged.
  - Copies entry in the retired list shape: `Decode` fails, the tiers come back nil, and the entry is flagged.
  - `harden-live` absent: its tiers default to `[0]`, which matches the template.
  - Equal one tier below `BilledTier`: the entry is removed and re-encoded. An emptied `gates` map encodes as `{}`, and the test's `cmp.Diff` against the empty map passed.
  - The tier-3 and several-tiers rows each fail if the check on `harden-live`'s tiers is dropped, so the flag test is not vacuous. The removal rows fail without the delete.

- No finding for `FocusPath`. The only changes are the rename and `RoundFacts` calling `shedadapters.FocusPath`. The build, vet and tier-0 gate on six packages are green. The remaining `focusPath` identifiers in `bouncer.go` and `prompt.go` are parameter names, not callers.

- LOW (no fix needed): re-encoding through `yaml.v3` may drop blank lines between entries in a hub's `gate.yaml`. Comments survive, and the values are unchanged.

**Tool calls**
1. `grep` for `ReinterpretedMarker`, `ConfigOpenMaps` and `harden-live-copies` across `internal`: to see whether other tests assume no production reinterpretation, and whether reconcile prunes `gates` entries. My context lacked both.
