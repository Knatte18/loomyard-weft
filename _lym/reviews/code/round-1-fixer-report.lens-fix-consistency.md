```yaml
lens: fix-consistency
outcome: reported
```

The change set is consistent. I found one stale twin, and it is outside the diff.

Findings:
- LOW, `internal/loomcli/smoke_test.go:4` (outside the diff): the comment cites `internal/reedcli`'s `smoke_lifecycle_test.go` as the smoke suite whose conventions loom follows. That file still exists on Linux. Only the test the F1 fix cites moved, to `smoke_lifecycle_windows_test.go`, so this is a general convention reference and still holds. No action needed.

Checks that came out clean:
- F1: `internal/reedengine/doc.go` now cites `smoke_lifecycle_windows_test.go`, which exists. The test it names is the one that moved there.
- F2: the new clause in the `PATTERN.md` `PATTERN-test-speed` entry, "a skip names a missing prerequisite, never the OS", matches `pattern/PATTERN-test-speed.md`. Its "No test skips by design" section (lines 111-118) says a remaining skip names a missing prerequisite. The entry is still one line and the background link is intact.

Tool calls:
1. `ls` of `internal/reedcli`, a grep for `smoke_lifecycle_test.go` references, and a grep for "skip" in `pattern/PATTERN-test-speed.md`. My context lacked the sibling files and the other references to the old filename.
