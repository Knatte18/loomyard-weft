```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens on the F1 change, commit 12c343d3, which removes the scratch-ban assertion from `internal/editdirective/editdirective_test.go`.

No findings.

- The doc comment on `TestDirective_ReadsTheEnginesCopyWithoutBanner` now matches the review's requested wording.
- Nothing else in the Go or markdown tree references `scratchBuildBan`.
- The ban sentence survives only in the two stencil copies, `contracts/stencils/engine/claude/edit-directive.md:19` and `contracts/stencils/engine/codex/edit-directive.md:19`. They are identical, so the twins agree.
- The remaining assertions (engine tool wording, marker, leading comment, trailing newline) are unchanged.
- The diff touches only the test file.

Tool calls:
1. `git show` and `git diff` on 12c343d3, plus a grep for `scratchBuildBan` and the ban sentence: my context had the edit but not the committed diff or the other places the text appears.
