```yaml
lens: fix-consistency
outcome: reported
```

Findings (fix-consistency):

1. **MEDIUM** — `_lym/landing/summary.md`, `## Tests` section. The first two `TestCheckNewGitEntries` bullets still describe the old rule.
   - "an entry seen alone fails the package" and "an entry seen beside other binaries keeps the code and names them" are the old rule.
   - The new rule fails a package only when `ownInDir` is true.
   - The same section lists `TestClassifyProcesses` as covering only "other test binaries, descendants, and which work in the directory". It now also pins `ownInDir`, the looping and missing parent chains and the argv cut.
   - Your edits updated the "What changes", "Blame" and "Known gap" bullets but not this section. The old and new rules now sit side by side in one document.

2. **MEDIUM** — commit d745ce69 carries no `Loosened-Check:` trailer, although it loosens existing assertions.
   - The rows `EntrySeenWhileNoOtherTestBinaryRanFailsAndIsRemoved`, `EntrySeenInFirstDirectoryWhileNoOtherTestBinaryRanFails` and `RunCreatingAnEntryWhileAloneFails` were reworked.
   - Their old inputs (attributed, no others) now keep code 0, where they failed with code 1 before.
   - The review's "Focus outcomes" says earlier commits carried a trailer per loosened check, and instruction-fix.md requires one for each widened or removed assertion in a pre-existing test.
   - The commit message ends with a plain sentence, not a trailer.
   - The original cause was measured: 22 gate runs between 2026-11-06 and 2026-11-18 in which the old rule blamed a package that did not write the entry.
   - The fixer should add the trailer lines in a later commit, with no change if need be, and should not rewrite history.

3. **LOW** — `internal/testkit/tmuxkit/gitentry_test.go:1`. The file header line is about 128 characters, over the 120-character limit in the prose rule. It also says "holds the tests of … the report, the watch and the process classification". The line should be split at the colon.

Checked and consistent:
- `tmuxkit.go`'s package doc, `PATTERN-test-isolation.md` and `gitentry.go`'s `describeGitEntry` agree on the new rule.
- `gitEntrySighting` and `checkNewGitEntries` docs match their code.
- The grep for old-rule wording ("ran alone", "no other test binary", "binary that every") found no other stale occurrence in Go or markdown outside `_lym/reviews`.

Tool calls:
1. Ran `grep` over `*.go` and `*.md` for the old-rule phrases. My context did not show other files that might repeat the old wording.
