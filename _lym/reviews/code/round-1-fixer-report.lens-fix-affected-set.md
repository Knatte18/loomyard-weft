```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. For F1 and F2 I reran the finding's search (every site that describes the "covers nothing" rule or the old judging, and every `spliceReport` caller). F3–F6 are single-site test gaps, not sets, so I left them unchecked.

**Finding 1 (LOW), `internal/testcover/doc.go:13`.** This is a twin of the sentence the fixer widened in `pattern/PATTERN-test-speed.md:263`, and it was left as it was.
- It reads "A test that is skipped, failed or covers nothing is not judged."
- `cmd/testtiming/main.go:78` and the PATTERN entry now say "covers nothing of its own package". Only the package doc of `internal/testcover` still states the old rule.
- Fix: widen the sentence in `doc.go` the same way, or confirm the package doc is meant to describe the generic `Judge`, which has no package view. `Judge` itself only sees whatever blocks the caller passes, so either reading is defensible. A reader still sees two different rules for the same list.

**Finding 2, no finding.** The only non-test caller of `spliceReport` is `cmd/testtiming/redundancy.go:280`. It passes the same `tags` that `renderReport` uses, so the refreshed header matches. The `splitReport` header is now discarded in `spliceReport`, and no other caller of `splitReport` exists.

**Unchecked:**
- F3, F4, F5, F6 (single test gaps, no set).
- Whether `docs/benchmarks/test-redundancy.md` still carries the stale header until the next `-pkg` splice. The review allowed regenerating the report or fixing the code, and the fixer chose the code.

Tool calls:
1. One grep across `.md`, `.go` and `.yaml` for "covers nothing", "covered by other tests", "test-redundancy" and "-redundancy". I needed every site that states the old rule, and my context held only the files the findings named.
2. A second grep in the same call for `spliceReport(` and `splitReport(` callers. My context did not show callers outside the files read.
