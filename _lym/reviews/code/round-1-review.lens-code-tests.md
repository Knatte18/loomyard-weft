```yaml
lens: code-tests
outcome: reported
```

**Code-tests lens: the test that pins each behaviour the diff adds**

- **Launch line sets `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL='1h'` (`buildLaunchCmd`).** Pinned by `TestBuildLaunchCmd`. Every row compares the whole line by exact equality against the `posixEnvLead`/`pwshEnvLead` literals (command_test.go:29-30), which now carry the variable, or against an inline literal that does (row at :93). Both the pwsh and posix forms are covered (posix rows at :77, :253, :266 and others). Dropping the assignment, renaming the variable or changing its value fails every row.
- **Resume line sets the same assignment (`buildResumeCmd`).** Pinned by `TestBuildResumeCmd`. Its rows use the `pwshEnvLead` literal or the inline literal at :872, under exact equality. `TestBuildResumeCmd_NoticePosix` (:913-917) builds its want from `posixEnvLead`, so it covers the posix form.
- **Open launch: both the fresh and the resume line export the subagent TTL, after the role's TTL export (`OpenLaunch`).** Pinned by `TestOpenLaunch` through `openLines` (:383-386). `openLines` builds the expected lead from the literal `"CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL"` and `"1h"`, not from the constant under test. It fails unless both `Cmd` and `ResumeCmd` open with that lead, so a missing export, the wrong order or a missing line fails it.
- **The wording of `doc.go`, `template.yaml`, `docs/overview.md` and the comments.** Static prose, exempt from this lens.

Every new behaviour has a row that fails when it breaks, and every expected value is an independent literal, so none of these tests passes by construction. I found nothing at any severity.

One thing outside this lens: `TestOpenLaunch`'s doc comment (command_test.go:395-397) still lists only "the export of the role's prompt-cache TTL" among the things it pins. That belongs to the affected-set lens or the handler.

Tool calls:
1. Grepped command_test.go for the env-lead and `openLines` uses and for each test's shell dialect. My context did not hold the line numbers of the rows or which dialect each row runs, and I needed both to confirm every row compares against the new lead.
