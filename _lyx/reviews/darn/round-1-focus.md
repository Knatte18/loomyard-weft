---
round: 1
focus:
  - "internal/reedengine/lock.go `withOpLock` and `repinStaleBindingsLocked`: does every blocking reed op re-pin a stale server before its step, and can the extra `display-message` or re-pin break a path that runs with no server or with the lock held re-entrantly?"
  - "internal/reedengine/bindings.go `bindingsStale` and the stamp written by `pinBindingsLocked`: does the rule match the board entry (a different build's bindings re-pinned, follow's default unchanged), including absent, unparseable, equal and newer stamps and a binary with no VCS stamp?"
  - "Docs and budget: do `doc.go`, the `bindings.go` header and `cmd/lyx/testdata/test-budget.yaml` agree with the diff, and is each claim in `_lyx/landing/summary.md` borne out?"
  - "Tests in `internal/reedengine/bindings_test.go` and the adjusted `set-option` counts: does each new behavior have coverage without duplicating an existing test?"
---

Seed focus from the board entry (#646) and the change description.
