```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round's change set (F1–F8).

**Findings**

- **LOW** — `internal/runprofile/template.yaml:3` and `docs/overview.md:1334`. These are twins of the F6 comment fix, left as they were. Both describe `raddle_auto_approve` as filing "without operator approval" and say nothing about split-perch runs, where the key is never read. Evidence: the F6 change to `internal/loomengine/template.yaml:5` and `internal/loomengine/config.go:612` now says the key is "never in a run whose recipe rows hold a split perch". `contracts/specs/raddle-proposal-spec.md:68` and `internal/loomshed/doc.go:247` already state the perch case. The finding is only that these two mentions don't carry it. The runprofile comment is a short setting description, so the omission is arguable and not clearly stale.

The other changes agree with their surroundings:
- **F7 heading.** Other "entries the operator approves" mentions are in `contracts/recipes/raddle-recipe.yaml`, `recipes.go`, `arbiterprompt.go`, `shedrun/seed.go`, `loomrecipe/doc.go` and a `recipekeyed_test.go` string. They describe the raddle recipe, which is never in perch mode, so they stay true.
- **F8.** `RowsHoldSplitPerch` is the only comparison of `artifact_paths` against the proposal path. `HoldsSplitPerch` goes through it.
- **F5 test rows and `firstMode`.** The error text names "split perch" and "raddle_auto_approve", matching the production messages.

**Tool calls**
1. A grep for the old F6 comment text, the old F7 heading text and the `RaddleProposalRel` or `FromSlash` uses. I needed it to find twins elsewhere in the repo.
2. A grep for files naming `raddle_auto_approve`. I needed it to find other documentation of the key.
3. A grep for the key in `raddle-proposal-spec.md`, the runprofile template, `docs/overview.md` and `loomshed/doc.go`. I needed it to read how each describes the key.
