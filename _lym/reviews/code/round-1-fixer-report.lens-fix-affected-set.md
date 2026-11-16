```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes cover every member of the sets the findings name (the `pr open` refusal descriptions, the `landWorld` seams, the `judgeOpenable` paths).

- **F1 (refusal descriptions).** I searched the other places that describe `pr open`'s refusals: `docs/overview.md`, `internal/landingshed/doc.go`, `describe.go` and `role-beams.md`. None of them lists the refusals, so no sibling is left stale. The `prUpdateCmd` help says it "refuses what `lym beam pr open` refuses of the entry, the owners, the run and the ready record". That enumeration is limited, so it needs no change.
- **F2 (rows pinning behaviour).** Both new rows cover the paths the review named. I found no third unpinned path in `judgeOpenable`.
- **F3 (fake seams).** `landWorld.deps` is the only fake that builds `ParentConflicts` and `Tasks` in `internal/beamcli`. Both now return an error field. `adoptcli`'s `Tasks` fake belongs to a different deps type and is out of scope.

One point I did not check. I did not rerun the full-package gate after the commits. The parent's gate run passed before the commits and nothing changed after it.

no findings

Calls made:
1. Grep for descriptions of the `pr open` refusals across the repo and `ParentConflicts` and `Tasks` fake sites. I needed it because my context lacked other docs and other `landDeps` constructions.
2. Grep over `docs/overview.md`, `internal/landingshed/doc.go`, `describe.go` and `role-beams.md`. I needed it because my context lacked what those files say about `pr open` refusals.
