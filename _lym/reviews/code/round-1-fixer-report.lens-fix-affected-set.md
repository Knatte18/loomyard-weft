```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1: the only production caller of `startAction` is `internal/fronttask/hub.go:107`.
- F2: the only production caller of `fronttask.File` is `internal/adoptcli/fixloop.go:258`; no other site reads the old "record the action" error.
- F3: the other `pointer.Reason` assignments in `darnwrite.go` always end in non-empty text.
- F4: `resumeHaltedChildren` is the only `ChildStatus` reader in `adoptcli`.
- Not opened: the condition-stop mentions in `internal/loomshed/doc.go` and `internal/darnengine/doc.go`.

Tool calls: two greps for callers and condition-stop users.
