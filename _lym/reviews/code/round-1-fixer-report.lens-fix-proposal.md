```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- F1: round-trip row and RunRecord{} omitted-key check fail if the field, tag or omitempty is dropped. The wiring assertion was conditional; internal/gatecli/tierrun_test.go holds only TestRetryOnShortage and TestFailedRunLabel, so tierrun.go:115 stays unpinned, which follows from the reviewer's own conditional.
- F2: a bare HasPrefix match would drop TestA, so the row fails against that regression.
- F3: the row fails if the no-failures branch of tierNotice dropped the log line; it breaks no other row.

Tool calls: 2. Read tierrun.go 95-125; grep of tierrun_test.go.
