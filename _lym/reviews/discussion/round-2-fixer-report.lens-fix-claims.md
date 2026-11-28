```yaml
lens: fix-claims
outcome: reported
```

Task: check that every factual claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

Claims checked:
- D4 sub-bullet, "any other pair" re-seed, `lym loom goto --to Discussion-Write`, then `lym loom start`, and the matching Fact: holds. The recipe is read from the seed on every verb (`internal/loomcli/arm.go` `resolveRecipe`). `shedengine.Status` has no recipe field. Tapestry holds Discussion-Write before Record-Seed, so goto moves back. Goto leaves the run paused, and `lym loom start` handles a paused halt (`internal/loomcli/start.go:154`). The bootstrap commits the seed on every call (`sharedbootstrap.go:158-163`).
- Fact, "a motif run commits only on the records side": holds. Its commits are `CommitAnchoredPaths`, which commits in the run-records worktree (`internal/fabricengine/commitweftpaths.go:77`, `internal/loomcli/wiring.go:506,575`).
- Fact, Remove's local destructive gate and its origin landed-work gate with `Remote: true`: holds (`remove.go:307`, `:366`; `battencli/wire.go:451`).
- D7, "kept as the operator's on later reconciles": holds whether or not the base has taken the template's jacquard block (`follow.go:167-171`).
- D7, the reinterpretation runs after the follow: holds (`walk.go:250-266`).

Findings:

- MEDIUM. Changed text: D5, "A motif run commits no work to its task branch, so its teardown deletes the branch locally and on origin, and the start that follows the clear meets no leftover branch." The teardown deletes nothing when the pair is not level. When `deps.Level` reads false, Worktree-Teardown ends Done and keeps the pair standing (`internal/battenshed/teardown.go:131-133`). Level is false while origin lacks the pair's tips or a landing is still owed (`internal/battencli/wire.go:1790`). In that case the branch, and the pair, are still there when the clear runs. The sentence holds only for a teardown that removed the pair, so it needs that condition or a way forward for a pair that is still standing.

Further calls:
1. `sed`/`grep` of `status.go`, `wire.go` and `start.go`, to confirm Status has no recipe field and to see Level's wiring and how start handles a halted state. My context had only part of the Status struct.
2. `sed` of `start.go:138-166` and `wire.go:1786-1800`, for the parked states start resumes from and Level's contract. My context did not have either.
