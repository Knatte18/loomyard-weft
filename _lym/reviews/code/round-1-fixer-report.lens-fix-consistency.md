```yaml
lens: fix-consistency
outcome: reported
```

Task restated: check the fix change set (F1–F10) for stale siblings and twins left unchanged.

**Findings**

- **LOW** — `internal/loomcli/prreviewgoto.go:1` (file header, unchanged by the fix).
  - The header says the file "clears a PR-Review review's recorded autonomy and session when a goto moves the run onto the row".
  - F1 made `ClearReviewSession` also clear `ParentReviewAsked`, the ask mark.
  - The header now omits that field. A reader would notice, but nothing is wrong in behaviour.
  - Evidence: `internal/pullrequest/record.go:91` lists `ParentReviewAsked`, and the header at `prreviewgoto.go:1` does not.

No other inconsistencies found:
- The three `RelaunchRequest{` sites in non-test code are the new `loomcli` builder and the two in `battencli/drive.go`. The `battencli` ones are covered by the new `haltedRowGuard` comment.
- `orchcli/notify.go` has no remaining reference to the old unexported `withRunAutonomy` name.
- The `ParentReviewAsked` field comment in `record.go` still matches what the clear does.

**Tool calls (2 of 6)**
1. A `grep` for `RelaunchRequest{`, `ParentReviewAsked` and `ClearReviewSession` across `internal`, `docs` and `contracts`. My context did not show every constructor site or doc that mentions the cleared fields.
2. A `grep` for the callers of `NotifyRunFrom`, `NotifyParent` and `WithRunAutonomy`, to confirm the rename left no stale name.
