```yaml
lens: fix-affected-set
outcome: reported
```

Checked: the stale-wording and changed-signature sets for F2, F4 and F6. One stale site remains.

**LOW** — `_lym/landing/summary.md:53`, Tests section.
- The bullet "an entry seen alone fails the package" states the old rule. The fix rewrote the "What changes" and "Design decisions" bullets but not this one.
- Under the new rule, an entry seen with no process of this package in the directory keeps the code. The tests now say so: `EntrySeenWithNoOtherTestBinaryAndNoProcessOfThisPackageInItKeepsTheCode` and `EntrySeenWhileAProcessOfThisPackageWorkedInItsDirectoryFailsAndIsRemoved`.
- Line 54, "seen beside other binaries keeps the code", is also incomplete. A sighting beside other binaries with an own process in the directory now fails (`EntrySeenBesideOtherTestBinariesWhileAProcessOfThisPackageWorkedInItsDirectoryFails`).
- Line 62 omits the own-process-in-directory result of `TestClassifyProcesses`, and the Tests section does not mention the two-sightings row.

The other sets are covered:
- **Signature callers:** `classifyProcesses` has one caller, `gitentry.go:227`, plus the test at `gitentry_test.go:396`. `checkNewGitEntries` has callers at `gitentry.go:84` and `gitentry_test.go:204`. Both match the new signatures.
- **`map[string][]gitEntrySighting`:** the old `map[string]gitEntrySighting` type no longer appears anywhere.
- **"Writer" claim (F4):** no remaining statement that a named binary is the writer, in code or `PATTERN-test-isolation.md`. The package doc, `PATTERN-test-isolation.md` and the Known-gap bullet in the landing summary are updated.

Tool calls:
1. `grep` for stale phrases and changed symbols across `.go` and `.md`: to find sites outside the files in my context.
2. `grep` for "seen alone", "beside other binaries" and "descendants" in the landing summary, the package and the pattern file: to confirm the summary's Tests section, which I had not re-read after the edits.
