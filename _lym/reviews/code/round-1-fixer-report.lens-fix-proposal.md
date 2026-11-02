```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F2, F3 were all fixed as proposed).

**F1, MEDIUM:** no finding.
- The re-sign join row exports `internal/b#Member` and the part's card re-signs `internal/b#Member`. The old `ExportMet` branch `case CardTypeEdit` matched `resign.Target == export`. The row therefore fails if that branch is restored.
- The Delete rows (join and part check) fail if `CardTypeDelete` is restored. The old `case CardTypeCreate, CardTypeDelete` ran `refsMatch` on the Delete refs.
- The part-check Delete row exports `internal/x/x.go`, which the worktree holds. `ValidatePart` does not run the export-existing check, and the gate passed, so the row stays at one blocking finding.

**F3, LOW:** no finding.
- The new gate subtest and the new validate-parts row fail if the root is empty or wrong. In both, `export-existing` comes from `os.Stat` under `worktreeRoot`.
- The validate-parts row uses `internal/x/x.go`, which the fixture worktree holds.

**F2, LOW:** one finding.
- Changed text: `internal/planparser/exportname.go`, the `CheckExportName` doc line "The file-unit rule is applied to every handle, since an export is always a Create declaration, whose unit half is the one a resolver reads."
- Summary: "an export is always a Create declaration" states something the code does not hold, because an export can also be met by a Rename New side.
- Evidence:
  - `ExportMet` (`internal/planparser/join.go`) has `case CardTypeRename` and admits `pair.New` as meeting an export.
  - The review's own premise names "a handle that a part creates only as a Rename New side".
  - Under `checkHandleMalformed`, that handle is not a Create claim, so `fileUnitRuleApplies` skips the file-unit rule. `CheckExportName` still applies it.
- The doc comment therefore justifies the divergence with a false premise. It should say the export is judged as a Create declaration by choice, and that a handle met only by a Rename New side is judged the same way.
- The finding's `fix` key only asked the comment to say this, and the fixer carried it out as written, so the wrong wording comes from the proposal. A reader trusting the doc comment would miss the Rename case.

Calls made: one, `git log -p` on `internal/planparser/join.go`, because the old `ExportMet` re-sign branch was not in my context and I needed it to check that the new re-sign row fails with the defect restored.
