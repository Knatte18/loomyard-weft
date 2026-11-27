```yaml
lens: code-correctness
outcome: reported
```

I reviewed the code-correctness lens: I ran each changed function and branch through concrete inputs.

**Cases run**
- `summaryparser.ValidateDescription` / `forbiddenTermFindings`
  - nil patterns: nothing checked. Correct.
  - Two terms on one line: one finding. Correct.
  - Missing file: returns before the term check. Correct.
  - A pattern whose leftmost match is empty, such as `lym|` or `(x)?lym`: `FindString` returns "", so a line holding the term is admitted. Wrong, NIT.
- `landingshed.LoadConfig`
  - `describe_forbidden_terms: ['(']`: refused, naming the entry. Correct.
  - `[]`: no patterns. Correct.
  - Empty `describe_review`: grammar not checked. Correct.
  - Timeout 0: refused. Correct.
- `Config.ForbiddenTerms` on a literal `Config`: nil. Correct.
- `describeReviewSeat`
  - Empty key: skip reason, writer's model. Correct.
  - Bad spec: the error names the bare `describe_review`, not `landing.describe_review` (code-affected-set's area).
- `describeReportGate`
  - Blank line then a verdict: pass. Correct.
  - No verdict: fail. Correct.
  - Missing report: error. Correct, since shuttle holds a turn end until the output exists.
- `describeReview.check`
  - No marker: pending.
  - `timeout` or `gate failed`: pass.
  - `not started: …` (skipped seat, written synchronously in `Run`): pass.
  - `done` with no report: pass. `done` approved: pass.
  - `done` with findings: fails once as writer-only, then passes. Correct in a single session.
  - After a re-attach: `sent` is new, so the old findings are sent again (see finding 3).
  - A review of a draft the writer has since rewritten: findings handed regardless (see finding 2).
- `describeReviewGateEntry` with `Final`: always passes; PassOnCap with 2 attempts. Correct.
- Gate order (`describeEntry` prepend)
  - Description gate fails: review entry not reached. Correct.
  - Description gate passes, reviewer still running: run held. Correct.
- Reviewer budget
  - Seats start together; the reviewer's 15 minutes include its `lym shuttle await` for the description.
  - A writer taking 15 minutes or more leaves the reviewer timed out, and the text passes unreviewed (finding 1).
- `MultiLLMProducer.mapOutcome`
  - Timeout of a started chair: Stuck with empty Path; `runAndCommit` skips the commit and the halt is blocked. Correct.
  - Chair `Gate` spans and attempts are dropped from the pointer (finding 5).
  - Not started: error. Correct.
- `timeoutDetail`
  - Zero times or limit: "unknown". Correct.
  - Seat limit preferred over table limit. Correct.
  - Elapsed is taken from the run state's creation time, so a re-attached run reports its whole age. Correct.
- `writerExit` timeout: the seat-limit lookup by name works and Path stays the seat output. Correct.
- `prText.matches` / `normalisedText`
  - `\r\n` and trailing spaces: equal. Correct.
  - Null body vs empty body: "" equals "". Correct.
  - Interior differences: an edit, after which GitHub holds the sent text. No loop. Correct.
- PR-Review `openPullRequest` (patching `entryStatus` after an edit): the first wake does not edit again. Correct.
  - New pull request with nil `entryStatus`: the first poll reads GitHub's text and finds it equal. Correct.
- `syncText`
  - Transient edit failure: swallowed and retried at the next poll. Correct.
  - Non-transient failure: halt. Correct.
  - Unreadable description: warned and left. Correct.
- `Finalize.setRecordedText`
  - Gets `body` (`finalize.go:402`), which is the PR-Review body plus the flaky section; that matches PR-Review's text when nothing was flaky.
  - Status error goes to `githubFailure`; edit error halts. Correct.
  - Beam runs are skipped. Correct.
- `loomcli` validate-description: a `DescriptionTerms` error is reported with an exit code. Correct.
- `entries_gate` `description`: terms are read when the row is built; an error is wrapped. Correct.
- `beamcli` recorders and gate: config is loaded per call; an error is returned. Correct.

**Findings**

1. MEDIUM, design — `internal/landingshed/describereview.go:455` and `template.yaml:8`.
   - The reviewer's timeout runs from the start of the table, alongside the writer, and covers its wait for the description.
   - A writer slower than `describe_review_timeout_min` (15, against the writer's 30) leaves a reviewer that timed out. `check` then passes the text unreviewed, with only a `logger.Info`.
   - The template comment says this value is "the longest the writer waits for its report". That is wrong: the writer waits only for whatever is left of the reviewer's 15 minutes.
   - Evidence: `describereview.go:455`, `:531-533`; `landing-template-describe-review.md:30`; `seatengine/engine.go` `Run`, which starts all seats together.
2. MEDIUM, design — `contracts/stencils/landing/landing-template-describe-review.md:30` and `describereview.go:523-555`.
   - The reviewer reads the description as soon as `lym shuttle await` sees the file. That can be before the description gate passes and before the writer's later rewrites.
   - `check` hands those findings over without asking whether the text changed since the review, and the rewritten text is never reviewed.
   - Moving the gate order (`entries_describe.go:53`) does not put the mechanical gate in front of the reviewer; it only orders the writer's own checks.
   - Evidence: `shuttlecli/await.go` (returns once the path exists).
3. LOW, design — `internal/landingshed/describereview.go:497-503`.
   - `sent` lives only in memory, and `describeEntry` builds a new entry on every call.
   - On a re-attach, a `done` marker and a findings report from earlier in the session send the same, possibly already fixed, findings to the writer again.
   - The PassOnCap contract says a request that must not repeat makes its own closure idempotent; this one does not.
   - Evidence: `shuttleengine/gate.go:111-113`; `resume.go:131-147` (a lost assistant keeps its written marker).
4. NIT, design — `internal/landingshed/describereview.go:428-430`.
   - The comment on `describeReviewGateAttempts` says the second attempt is "the margin a re-attached run starts with".
   - Failure counts are in memory and reset to zero on attach, so that margin does not exist.
   - Evidence: `shuttleengine/gate.go:111`.
5. NIT, design — `internal/shedadapters/multillm.go:210-211`.
   - The timeout Stuck pointer drops `GateAttempts` and `GateSpans` from `chair.Gate`. A run held on a pending gate until its deadline then reports no gate attempts or held span.
   - Evidence: `multillm.go:192-194` (the Done and gate-failed branches carry them).
6. NIT, design — `internal/summaryparser/description.go:64`.
   - `FindString` returning "" counts as no match. A configured pattern whose leftmost match is empty, such as `lym|` or `(x)?lym`, then admits a line holding the term.
   - `FindStringIndex` with a non-empty match, or `MatchString`, would not.

Tool calls:
1. `finalize_pr.go:110-175` and `finalize.go:140-175`, to check that the text Finalize sets matches PR-Review's body; `bodyWithCredit`, `completeSummary` and `landingTexts` were not in my context.
2. `shuttlecli/await.go` `awaitCmd` and the `shuttleengine` `StartedAt`/`EndedAt` fields, to learn when the reviewer's `await` returns and what elapsed time measures.
3. A grep for the `LandingMessage` and `mergeOnGitHub` call sites, to learn which text Finalize passes to `setRecordedText`.
