```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind the set-premise findings (F1, F6) and check the fix covers every member.

**F1/F6 (template keys documented at scope `diff`):** the fix covers the sites the review named, but one more site was missed.

- `docs/dotnet-hub.md:119-121` — **LOW** — The hub sample lists `webster-fix`, `darn-write` and `code-burler-fix` at `scope: diff`, the same stale-sample shape as F6.
  - It is a hand-written dotnet hub file that sets the scoper to `dotnet`, and coverage narrowing applies to the `go` scoper only. Under `dotnet`, `coverage` equals `diff`, so the file stays correct.
  - A reader comparing it to the template's new coverage defaults would still notice the mismatch. Either add a line saying the sample keeps `diff`, or leave it.
- `internal/gateslot/doc.go:204-205` — no finding. These are the migration's legacy-list output (`darn-write`, `code-burler-fix`, `webster-fix` with scope `diff`) and describe what `MigrateConfig` emits, not the template defaults.
- `docs/research/...` — no finding. Dated research reports are measurement records.
- `internal/gateslot/template.yaml:20` and `internal/gateslot/doc.go:103` already state coverage.

Findings F2, F3, F4, F5, F7, F8, F9 and F10 are test or code additions with no set premise beyond what the fix covers, so I left them unchecked. The `_lym/landing/summary.md` entry for F7 is not tracked in git, so I did not check it.

Tool calls made:
1. One `grep` for the template keys beside `diff` across `.go`, `.md` and `.yaml` files. My context held only the files the fixer read, not the repository's other docs and samples.
