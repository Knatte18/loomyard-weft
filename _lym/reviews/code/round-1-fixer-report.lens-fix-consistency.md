```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The new `pr open` help bullets and closing sentence match `internal/beamcli/doc.go:164`, `contracts/orchrole/role-beams.md:39-41` and the refusal strings in `judgeOpenable` (`pr.go:247,254`). The `pr update` help defers to what `pr open` refuses of the entry, the owners, the run and the ready record, so it stays consistent. The `pr_test.go` header comment already mentions the merge-task and conflict refusals. In `land_test.go` the new `tasksErr` and `parentConflictsErr` fields are used by the new rows. The `Tasks` seam is also used by `TestMarkReady`, where `tasksErr` is nil by default, so those rows are unaffected.

One note below the finding threshold: the new help bullets use `<task>` and `<slug>` placeholders, while their neighbours use `<slug>` only. This is stylistic and not a disagreement.

Calls made:
1. Grepped `doc.go`, `role-beams.md` and `cli.go` for twin refusal wording and `merge-parent` mentions, because my context did not show those lines.
2. Ran `git diff HEAD~2 --stat` to confirm the change set covers only the three files.
