```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the `backUpBaseline` fix (metadata-move rollback) in `capture.go` for agreement with its surroundings.

- **Doc comment on `backUpBaseline`:** the new clause says "when that move fails, the baseline directory moves back". The code does this for every failure except `os.ErrNotExist`, and a missing source is not a failure there, so the two agree.
- **`RunCapture` doc comment:** `capture.go:77` says "Its metadata directory moves with it, so no baseline is deleted or overwritten". That is still true and does not need the rollback detail.
- **`doc.go:126-128`:** the doc describes the move, the free-number rule and the leftover-metadata removal. The rollback adds no new contract here, so nothing is stale.
- **Caller:** `RunCapture` still sends any `backUpBaseline` error to `failCapture`. The joined error wraps both causes and the record reason uses `cause.Error()`, so it carries both messages.
- **Rollback and `MkdirAll`:** a successful rollback returns before `MkdirAll`. That is correct, because the baseline directory exists again.
- **`nextFreeBackup`:** it is untouched. After a rollback `baseline.N` is gone, so the number is free again and the next capture can reuse it.
- **Tests:** the review said no test row is needed, and the existing backup test at `capture_test.go:537` is unaffected.

Calls made:
1. `grep -rn` for backup, half-moved and `backUpBaseline` text, to find sibling docs and callers that my context lacked.
2. `Read` of `doc.go` lines 120-134, to see the `RunCapture` backup prose in the package doc.
