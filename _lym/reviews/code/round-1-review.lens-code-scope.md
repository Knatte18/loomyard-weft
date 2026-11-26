```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to the board entry's goal points.

The decision record holds no `Added after Discussion` entries, so the decisions are the goal points of the board entry under `## Goal`.

**Per decision: the changes that build it**

- **G1 (#248): report an interrupted or reboot-gone fixer as interrupted or `died`, not `timeout`.** C1 builds it.
- **G2 (#248): resume the session, or spawn a replacement that spends no budget.** C2 builds it.
- **G3 (#239): a died judge is relaunched or halts as an infrastructure failure, spending no bounce budget.** C3 builds it.
- **G4: the Claude input-box reader treats the post-API-error prompt as an empty box, so revive reaches the session.** C4 builds it.
- **G5 (#219): check the box detection against the done-reply pane, and retry the idle check briefly before declaring died.**
  - C5 builds the detection part.
  - The retry needs no change: `sendStartPrompt` already re-sends on a busy hold up to `startPromptBusyRetries` (`internal/shuttleengine/run.go:974-989`), and each send polls the idle wait.
  - Covered.
- **G6a: test the degraded-launch Warn.** C7 builds it.
- **G6b: the OOM notice can name an earlier kill or the wrong process, either when panes share a cgroup or when the journal line lands after the kill is counted.**
  - C6 builds only the shared-cgroup double-notice part.
  - Nothing builds the journal-lag part or the misattribution part. The summary says they are "left alone". See F1.
- **G6c: test the notice's delivery to the parent orch end to end.** C8 builds it.
- **G6d: check that the OOM step does not stall the watchdog loop.**
  - No change was needed: `go watchPaneOOMKills(...)` runs in its own goroutine (`internal/reedcli/watchdog.go:351`).
  - The check holds.
- **G6e: check that a test covers `removeSupersededStrands` recording a stop as interrupted.**
  - No change was needed: `TestAttach_RecordsASupersededRunningCandidateAsInterrupted` exists (`internal/shuttleengine/attach_test.go:375`).
  - Covered.
- **G6f: a resumed start that fails to come up retries fresh.**
  - No change was needed: `TestStart_FailedResumeRetriesAsAFreshSession` exists (`internal/shuttleengine/crashresume_test.go:127`).
  - Covered. Whether the test pins the claim is for the tests lens.

**Per change: the decision it traces to**

- **C1** traces to G1. It covers:
  - the reorder in `await.go` and `goneStrandReason`;
  - `DeadReasonInterrupted` in `record.go`;
  - the doc lines in `doc.go` and `await.go`;
  - the new and renamed rows in `await_test.go`.
- **C2** traces to G2. It covers:
  - `waitReplacingInterrupted`, the new `replace` parameter, `markSpawned` and `note()` in `rowgate.go`;
  - `budget()` in `rowstore.go`;
  - the new subtests in `rowgate_test.go` and the `fakeFixer.interruptedFirst` field;
  - the edits to `rowstore_test.go` and to the existing `rowgate_test` assertion, both forced by the rename from `count()`.
- **C3** traces to G3. It covers:
  - `judgeDiedError`, the died branch in `judgeCall` and the doc edits in `bouncer.go`;
  - `TestBouncer_JudgeCall_ADiedJudgeIsAnInfrastructureFailure`.
  - The removed died row in Degradations and the Harvest contrast switched to timeout are forced edits: died no longer degrades.
- **C4** traces to G4: `holdsOnlyAPIErrorPrompt`, `apiErrorRetryPrompt`, the `IdleSession` and `InputBoxText` doc and code, and the new `session_test` rows.
- **C5** traces to G5: the plain-capture fallback in `boxCapture`, `styledTailLines`, the `boxLosingStyledEngine` fake and two `session_test` rows.
- **C6** traces to G6b: the per-cgroup skip and doc in `planPaneOOMNotices`, plus the pid 7 row in `oomnotice_test`.
- **C7** traces to G6a: the `TheUserManagerCannotCapThePane` row in `panememory_test.go`.
- **C8** traces to G6c: `oomnotice_integration_test.go`.
- **C9** is forced by the darn-write gate's comment lint: commit `fb3c33890` rewraps comments and `7d3d8858d` moves `degrade`'s doc comment back.

No change lacks a decision, so there is no unrequested scope.

**Findings**

- **F1**
  - Severity: MEDIUM
  - Class: scope
  - Location: `internal/reedcli/oomnotice.go:75-102` (`planPaneOOMNotices`) and `paneOOMNoticeLine` at `:109-122`
  - Summary: the board's carried-over finding says the notice can name an earlier kill or the wrong process when the journal line lands after the kill is counted, or when panes share a cgroup. The diff fixes only the double notice per shared cgroup. The summary's "Design decisions" says the rest is "left alone": kills in a descendant cgroup that shift the journal index, and a scope name reused after pid wrap. The record has no `Added after Discussion` entry licensing that omission, so that part of G6b is still unbuilt.
  - Evidence: the board entry's Goal, "Review findings carried over", second bullet, in `_lym/discussion/decision-record.md`; `_lym/landing/summary.md` "Design decisions", fourth bullet; the diff hunk at `oomnotice.go:530-534`.
  - Fix: build matching of kills to journal lines per cgroup, so a descendant cgroup's kills and a reused scope name cannot shift the names the notice gives. If the parent wants the omission instead, it should record it with `lym loom decision add`.

Tool calls:
1. One grep for `TestAttach_RecordsASupersededRunningCandidateAsInterrupted`, `TestStart_FailedResumeRetriesAsAFreshSession`, the goroutine that starts the OOM watch, and `sendStartPrompt`'s retry loop. My context held the summary's claims about G6d, G6e, G6f and G5's retry, but not the code that would confirm them.
