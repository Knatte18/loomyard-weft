```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- F1 (`internal/reedengine/doc.go:485`): the cited path now holds the test. `internal/reedcli/smoke_lifecycle_windows_test.go:23` declares `TestSmokeRemoveLastStrandThenAddRunsTheNewCommand`. No other `.go` or `.md` file outside the benchmark report names the old location with that test. The `docs/benchmarks/test-redundancy.md:2118` mention is a measurement report, which the review rejected.
- F2 (`PATTERN.md:60`): the added clause "a skip names a missing prerequisite, never the OS" matches `pattern/PATTERN-test-speed.md:111-118` ("No test skips by design"; a remaining skip "names a prerequisite the host lacks"). It leaves the entry on one line, and the background link is intact.

Tool calls made:
1. `grep` for the test name across `.go` and `.md` files, `ls internal/reedcli`, and `grep -n -i skip pattern/PATTERN-test-speed.md`. My context lacked the post-move file listing and the background file's wording.
