```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1/F2: `Admit` has one caller, `internal/gitrepo/remoterefslock.go:30`. The in-flight descriptions in `internal/tenderengine`, `hibernateengine`, `tendercli` and `docs/overview.md` say "network operations in flight" and are still true of a lock-hold marker. They don't claim one marker per call.
- Both `hold.go` and `doc.go` were updated, and `PATTERN.md` carries the process-wide note.
- F3: no checks beyond the context. The `Committed` stub is the only one in `cleanup_test.go`, and I did not search other test files for `Committed` stubs. They are not at issue, because the finding names this file only.

Tool calls:
1. grep for `Admit` callers and "in flight" docs across `.go` and `.md`, to find sibling sites my context lacked.
