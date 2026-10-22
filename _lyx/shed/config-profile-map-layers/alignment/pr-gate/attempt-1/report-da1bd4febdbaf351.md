---
verdict: APPROVED
---

## Scope

Measured the diff of pull request 836 at `c8e2d4a35` against `main` (merge base `f1df06b57`) against board entry `config-profile-map-layers`.
The decision record has no `Added after Discussion` entries, so the board body is the measure.

- Issue #833, `--set` of a nested profile key: `internal/yamlengine/set.go` and `layers.go` make `SetValues` and `SetSparse` know `profiles.<name>.<rest>` for a path some template entry holds and write that one leaf under the template leaf's tag, leaving the profile's other keys to lower layers.
- Issue #834, a hub `profiles:` hiding the template's profiles: `batcher.ConfigSpec` and the `configreg` row declare `profiles` a per-name map in place of an open map, so the merge and the template fill step into it by name and into each profile key by key.
- The supporting changes follow from the declaration and stay inside the two issues:
  `configengine.ValidateLocal` and `ValidateMachine` admit a profile name the template lacks, judged against the template's entries;
  batcher's retired-key refusal names the highest layer holding the profile;
  `configsync`'s former batcher template value is narrowed to the one leaf the tuned defaults replaced, since reconcile now compares the per-name map leaf by leaf;
  `yamlengine` keeps marshalled output valid where a filled or emptied mapping carries a comment.
- The docs of `batcher`, `configengine` and `yamlengine` change in the same commit.
- The hub's own copies of `cautious` and `identity` are left to the operator; the entry says they can be removed once fixed and asks no removal of this run.

No change goes beyond the entry, and no point of it is left out.

## Open risks

The two open risks in the decision record, a layer no longer able to hide a template profile and a retired key attributed to the highest layer holding a split profile, are consequences of the entry's own goal.
Neither names a parent correction, so neither escalates.

## Overlap

- Peer `run-process-ownership` (row 15, this run row 21): its changed paths share no file or package with this diff. No overlap.
- `lyx loom peers` lists no landed parent commits.

## Superseded

None.
Note `batch-budget-300k-trial` refers to this fix, but it tracks the trial of the `wide` profile, which this run does not end.
