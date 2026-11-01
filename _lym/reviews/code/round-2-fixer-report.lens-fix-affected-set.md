```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Scope: F1's premise is a set, the sites under crucible that name a `//go:build tierN` tag. I reran its search.

- **After the fix:** `crucible` and `internal/testkit` have no `go:build ... tier2/tier3` text, no "tier2/tier3 tag" wording and no `-tags tier2/tier3` command left. The fix covered the four template sites the review named. It also covered `crucible/README.md:201`, a twin the review missed.
- **Test files:** no `_test.go` file in the repo opens with a `//go:build ... tier2` or `tier3` line. So `tiergate.Require(t, 2)` and `tiergate.Require(t, 3)` are the only live-test markers. That matches the new wording.
- **Matches outside `crucible`:** other `go:build tierN` and `-tags tierN` mentions remain in `internal/gatecli/doc.go`, `internal/planparser`, `internal/shellguard` and `internal/impactset`. They concern tier0 and tier1 tags, the gate's own `-tags` handling, or test fixtures. They are not about locating live tests in the crucible template, so they are outside F1's premise and I raise nothing.

Tool calls:
1. `grep` across the repo for `go:build tier` and `-tags tierN` forms: to find sites outside `crucible` that my context lacked.
2. `grep` over `crucible` and `internal/testkit` for tier2/tier3 tag text, plus a `_test.go` search for tier2/tier3 build-tag headers: to confirm no stale site remains after the edits and that no test file still carries the tag.
