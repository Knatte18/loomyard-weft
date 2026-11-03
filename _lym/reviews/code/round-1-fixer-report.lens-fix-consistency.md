```yaml
lens: fix-consistency
outcome: reported
```

Findings (fix-consistency):

1. **LOW**, `internal/burlerengine/assist.go:119`, the `composeAssist` doc comment. It still says it "returns the files' paths and an empty reason, or the reason the assistant cannot start". Since F2, a failed composition also returns file paths: `unstartedAssist` hands back the probe directory's `reviewFilesIn` paths. That contradicts the doc, which implies no usable files on failure. The paths are placeholders the seat never reads, so a reader would take them for real instruction files. The doc should say so, in step with the `unstartedAssist` comment.

2. **LOW**, `internal/loomengine/template.yaml:29` and `internal/loomengine/config.go:11`. The F3 change makes `assist_reviewer_timeout_min: 0` mean "take the whole capped room", and `CapAssistTimeout` now documents that. The key's template comment says only "capped so the assistant ends before the reviewer's own timeout", and the `config.go:11` header says only "refused below 0". Neither documents the 0 meaning, though 0 is the only value the change makes behave differently. This is a stale sibling, not a wrong statement.

No other twins left behind:
- The `"burler"` and `"burler-review"` literals are gone from non-test code: `review.go` and `assist.go` use `burlerDirName`, and `wiring.go` uses `ReviewRolePrefix`.
- No other `reviewRolePrefix` references remain.
- The `docs/overview.md` additions match the key names in `config.go` and the template.

Tool calls:
1. One `grep` for `assist_reviewer_timeout_min` across `.go`, `.md` and `.yaml` files, plus the `composeAssist` doc line. My context lacked the key's other documentation sites (template and config header) and whether the doc line had changed.
