```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for places where a sibling or twin text disagrees with the change. This is the fix-consistency lens.

No finding in this pass is BLOCKING or MEDIUM. The doc and contract twins are consistent: `docs/overview.md` (gate paragraph and the lint-verbs paragraph), `pattern/PATTERN-gate-slot.md`, `contracts/stencils/webster/webster-body-implementer.md`, `internal/gatecli/doc.go` and the loomcli verb docs all agree with the new behaviour. No leftover of the old `... more lines; the logs named above hold them` text, or of "every card gate runs" lint wording, remains outside `docs/research`, which is historical.

**Findings**

1. **LOW** — `failureReportLines` const comment, `internal/gatecli/failurereport.go:27`.
   - It says "the logs it names hold the rest".
   - The fix makes that false for a lint step, which has no log; `cutLine` now says so.
   - Evidence: `cutLine` emits "`<lint>` has no log, so re-run `lym loom lint-…`".
   - Reword the const comment to match.

2. **LOW** — `renderFailureReport` doc comment, `internal/gatecli/failurereport.go:~165`.
   - It still says only "a bare file name prefixed with its package's directory".
   - `qualify` now also prefixes run-directory-relative paths (`./x.go`, `dir/x.go`) for failures outside any test, which `doc.go` documents.
   - Update the sentence.

3. **LOW** — `runStandIn` doc, `internal/gatecli/test_integration_test.go:50-51`.
   - It describes the build invocation as only writing `-o` content and exiting with `helperBuildExitEnv`.
   - The shared build now also prints `helperBuildFailureLine` on a non-zero exit (`sharedBuild && code != 0`), and the F2 assertion depends on that.
   - Add that to the helper role comment.

4. **LOW** — `cutLine` is passed through `capLine` (`failurereport.go`, the `lines = append(lines, capLine(cutLine(cut)))` line).
   - With several cut runs, the 300-byte cap can truncate the line.
   - The re-run hints sit last, so they are the first to be lost, the opposite of what F1 needs.
   - The new test's line is about 200 bytes. Three or more cut runs, or two lints plus a logged run, would exceed 300.
   - Don't cap the cut line, or put the hints before the per-run counts.

Further tool calls: 2.
1. `grep` over `*.go`, `*.md` and `*.yaml` for old cut-line text and "card gate runs" wording. Context held no view of files outside `gatecli`.
2. `grep` over `docs/overview.md`, `pattern` and `contracts/specs` plus a `git show --stat`. I needed the `overview.md` gate paragraph and the lint-verb docs, which my context lacked, and the commit's file set.
