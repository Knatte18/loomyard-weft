```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each set-premise finding's search to see whether the fix covers every member.

Findings checked: F3, F5, F6, F7. F1, F2 and F4 are test additions with no set premise, so I left them unchecked.

**MEDIUM — `pattern/PATTERN-test-speed.md:118` contradicts the line F6 added at line 119.**
- The line still reads "A narrowed gate's guard step runs every marked test by name under its file's tier, whatever the impacted set."
- The new line directly below says a guard test that also carries `//lym:watch` is left out of a narrowed gate's guard step.
- F6's review text calls the section wrong, and the unqualified "whatever the impacted set" claim is still there. A reader sees two lines that disagree.
- Suggested fix: qualify line 118, for example "except a test that also carries `//lym:watch`, see below".

**LOW — `internal/impactset/doc.go:42` is untouched by F6.**
- F6's review says `doc.go` "already says so", and that is true of the gatecli doc.
- Line 42 reads "The gate's guard step runs them under the command of each test's tier". It does not mention the watched-guard drop.
- I did not open the lines around it, so treat this as unchecked.

No finding for F3, F5 or F7:
- **F3:** Nothing outside `internal/gitexec` touches `armDeadline` or `SetDeadlineForTest`. The exception sentence in the pattern covers that package.
- **F7:** The refusal text appears only in `cli.go:240` and at `impactset/guards.go:199`, which is the source error. No test pins the old `%v` form.
- **F5:** Another `sleep 30` use, `run_tied_integration_test.go:81`, already follows the `sleep 30 &amp; … ; wait` shape. `testmain_test.go:36` uses a bare `sleep 30` and is not part of this finding.

Calls made:
1. One Bash call with four greps, one per finding (F6, F7, F3, F5). My context did not hold these repo-wide searches.
