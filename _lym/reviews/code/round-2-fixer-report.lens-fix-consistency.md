```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (`follow.go`, `follow_test.go`, `yamlengine.md`) for consistency with its siblings.

No findings.

- **Doc twins agree.** The `FollowTemplate` doc comment and the `yamlengine.md` line both now say `base` is not read when `haveBase` is false. The `ErrInvalidBase` comment ("does not parse or is not a mapping") still holds, because the error is now raised only when a base is read.
- **Caller agrees.** `configsync.go:1046` passes `templateBasePresent && !fileAbsent` as `haveBase`. The `FileError` wrap at `:1047-1048` and the `ReconcileHubWideAt` comment at `:935-937` already describe the base being used only when the board file is present, so neither needed a change.
- **Existing corrupt-base tests keep failing as asserted.** The `configsync_test.go:997` row has `hubFile: template`, so the file is present and `haveBase` stays true. The `ensure_integration_test.go:990` row commits a corrupt base onto a stale hub. That hub's board file is presumably present, so `haveBase` stays true and the row still reaches the error. That is an inference from the fixture helper, and the fixer's gate run covered only `./internal/yamlengine`. The `hubreconcile` and `configsync` tests are the ones to run if you want it confirmed.
- **New row is distinct.** The `follow_test.go` row has `haveBase` false with a non-mapping base and expects no error. No other row covers it, and the trailing list-document check in that test targets `existing`, not `base`.

Further tool calls (4):
1. `grep` over `configsync.go`, `configcli/doc.go` and `hubwide.go` for corrupt-base and template-base wording, plus `git show --stat HEAD`. My context lacked these sibling texts and the commit's file list.
2. Read `configsync.go:932-947`. My context lacked the `ReconcileHubWideAt` doc comment.
3. Read `configsync.go:1036-1059`. My context lacked the caller of `FollowTemplate`.
4. `grep` and two Reads of the corrupt-base tests in `configsync_test.go` and `ensure_integration_test.go`. My context lacked the rows whose behavior the change could affect.
