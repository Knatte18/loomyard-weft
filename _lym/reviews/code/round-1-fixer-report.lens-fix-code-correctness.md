```yaml
lens: fix-code-correctness
outcome: reported
```

Task: self-check of the fix's code changes for correctness.

**LOW**: `internal/loomcli/prreview_mail_integration_test.go`, `TestPRReviewGateFailureMail`. The expected head comes from the same seam the row runs on.
- `head, err := wired.TaskHead()` supplies the expected mail text, halt reason and `MailedHead`, while `prReviewMailDeps` sets `deps.TaskHead = wired.TaskHead`.
- A `TaskHead` seam that read the wrong commit, such as the records head, would still pass all three rows.
- An independent read such as `gitkit.RevParse(t, pair.WorktreePath(), "HEAD")` would catch it. The review's F1 fix asked for the assertions to come from "that real head".

I found nothing else:
- **`LiveStrandNames`:** `reedEngine` is a non-nil pointer assigned once (`wiring.go:686`), so binding `reedEngine.Status` early is safe, and line 1270 already does it. The new assertion in `TestSessionReads` covers the dead, retiring and table-failure inputs.
- **`replayedSessions`:** both callers in `envkit.go` take the fourth result.
- **`placeholdCall`:** a missing `{hub}` binds `""`, which `placehold` skips. The extended test fails if every binding is applied to stdin again, or if `bindCall` stops binding `Stdin`.
- **`TestIdleSession_BindsTheBareNameOfAShellTheHostLacks`:** it fails if the fallback is removed, and also if the shell override is ignored.
- **Mail test:** the push counter and whole-record comparison cover the checks F3 named. Its subtests ran and passed in tier1.

Tool calls: one grep of `internal/loomcli/wiring.go` for how `reedEngine` is bound, because my context held only lines 874–915.
