```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: no BLOCKING or MEDIUM findings. One LOW.

**[LOW] `innerRunEntry` and `runShedDuration` disagree on whether a non-positive `notice_quiet_min` is valid**
- Changed text: `runShedDuration` in `internal/beamcli/dobbydeps.go` now falls back to 30 minutes for a count that is not positive. `innerRunEntry` in `internal/shedrecipe/entries_batten.go:89,118-124` and `TestInnerRunEntry_NoticeQuietKey` in `internal/shedrecipe/entries_batten_test.go:268` still validate the key as "non-negative", so zero passes validation.
- The contracts agree in behaviour: zero is accepted and means the default. The wording does not say so. The `innerRunEntry` doc says only that the key is "validated as a non-negative int" for the hub wiring. `batten-recipe.yaml:78` has no comment on the key, so no reader is told that zero means 30 minutes.
- Impact: documentation only. The reviewer's F5 offered "say it" as the alternative to the code change, and the code change was chosen.

Checked and consistent:
- `ChildReading.RespawnCap` replaced `RespawnCapped` everywhere. A grep finds no remaining `RespawnCapped` reference.
- The `RespawnCap` const in `battenshed` is the only cap in the notice path. The `dobbyengine` copy is gone.
- Every caller of `RespawnCapReached` takes the new `statusModified` parameter: `internal/battencli/dobbynotice.go` and the `internal/battenshed/innerrun_respawncap_test.go` test.
- `internal/battenshed/doc.go:89` still reads correctly, though it says "reads the respawn count" and does not mention the status-write reset.
- `resetText` and `resetStamp` were deleted from `battenshed`. `dobbyengine` keeps its own copies, and nothing else in `battenshed` referenced them.

Tool calls:
1. grep for `notice_quiet_min`, `RespawnCapped` and "respawned the cap": to find twin references and stale docs outside the changed files.
2. `sed -n 70,82p contracts/recipes/batten-recipe.yaml`: to see whether the recipe comments the key, which my context lacked.
