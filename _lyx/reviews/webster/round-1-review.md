---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/shuttleengine/claudeengine/session.go:79
    summary: The labelled-rule shortcut in isBoxRule also applies to the bottom-rule walk, so a draft continuation line shaped like a labelled rule closes the box early and IdleSession reports idle on a non-empty box; session_test.go has no near-miss case
  - id: F2
    severity: LOW
    location: internal/orchcli/smoke_live_test.go:360
    summary: TestSmokeOrch_SoftCycle accepts the bare word background as proof the handoff names the shell, but the soft stencil itself says background shell, so the assertion does not show the handoff names the test's shell or that the background-shell turn, not the start turn, started the cycle
  - id: F3
    severity: LOW
    location: internal/shuttleengine/spec.go:62
    summary: The new AllowAgentTool, PermissionMode and ResumeSessionID doc comments and the rewritten Interactive lines, plus run_resume_test.go's new file header, are hard-wrapped at a fixed column mid-sentence, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F4
    severity: LOW
    location: internal/shuttleengine/claudeengine/resume.go:1
    summary: resume.go has no file-header comment, while every other production file in claudeengine opens with one naming what the file implements; resume_test.go likewise lacks one
  - id: F5
    severity: NIT
    location: internal/shuttleengine/claudeengine/resume.go:47
    summary: A transcript stat error other than not-exist is refused with a message saying there is no transcript, which misstates the cause when the file exists but cannot be statted
  - id: F6
    severity: NIT
    location: internal/shuttleengine/claudeengine/claudeengine.go:62
    summary: Prepare's doc comment still says it validates spec.Effort and spec.Model before writing artifacts, but it now also validates PermissionMode and ResumeSessionID there
  - id: F7
    severity: NIT
    location: internal/orchengine/watcher_test.go:975
    summary: The DEFER test reads its fresh turn end at the same instant as the DEFER, so nothing pins that the DEFER turn end alone never re-qualifies once soft_idle_s passes
---

Range: `product.parent` is `main` (from `_lyx/shed/orch-live-ready/status.json`), and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`, which spans the thirteen card commits.
`go build ./...`, the plan's `go test -tags integration` line and `go vet -tags smoke` all pass.

### [LOW] isBoxRule's labelled-rule shortcut also closes the box from below

**Location:** internal/shuttleengine/claudeengine/session.go:79, session_test.go:39

**Issue:** Commit 13 added a shortcut: any trimmed line starting with `───` and ending with `─` counts as a rule.
`IdleSession` uses `isBoxRule` both for the top walk and for the bottom walk.
Claude labels only the top rule, but the shortcut also accepts a line below the caret.
I probed this through a `go test -overlay` file.
A capture of `rule`, `❯ ` (empty first line), a draft continuation `  ─── note ─`, more draft text and the real bottom rule reports idle.
The walk down stops at the draft line, and the only interior line is the empty caret line.
The pre-existing check already accepted a draft line made only of rule glyphs, so this is a widening rather than a new class of failure.
It is still a fail-open path in a probe documented as fail-closed, and a false idle types the handoff request or `/clear` onto an operator's draft.
`session_test.go` covers only the labelled rule itself, with no near-miss case.
The change also falls outside card 13, whose card says no production code changes.
The orchengine doc records why it was needed (the probe never passed on a named session), so the departure is recorded and the fix itself is justified.

**Fix:** Accept the labelled shape only for the top rule.
The bottom walk keeps the strict glyph-only check.
Add near-miss cases: a labelled-rule-shaped draft line below the caret is not idle, and a line carrying a label with no trailing rule glyph is not a rule.

### [LOW] Soft-cycle smoke asserts the handoff names the shell too loosely

**Location:** internal/orchcli/smoke_live_test.go:360

**Issue:** The check passes when the handoff contains `softbg`, `sleep 900` or merely `background`.
The soft stencil asks the session to list every "long-lived watcher (background shell, Monitor)".
A handoff saying there is no background shell would therefore still pass.
Card 13 asks that the handoff names the background shell, and that the background-shell turn's end, not the start prompt's, starts the soft cycle.
Only a handoff naming the test's own shell (`SOFTBG` or `sleep 900`) shows both.

**Fix:** Drop the bare `background` alternative, so the handoff must name `SOFTBG` or `sleep 900`.

### [LOW] New Spec field docs are fixed-column wrapped

**Location:** internal/shuttleengine/spec.go:62-88; internal/shuttleengine/run_resume_test.go:1-4

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks for new and changed comments.
It also says existing fixed-column-wrapped comments are legacy, not a convention to match.
The three new field docs and the changed `Interactive` sentences break mid-phrase at a fixed column, for example "lets this one run use the Agent tool for / every subagent type".
The new `run_resume_test.go` header does the same.

**Fix:** Rewrite those comments one sentence per line, breaking further only at an independent-clause boundary.
Leave the untouched legacy lines of `Interactive` as they are.

### [LOW] resume.go has no file-header comment

**Location:** internal/shuttleengine/claudeengine/resume.go:1; resume_test.go:1

**Issue:** Every other production file in `claudeengine` opens with a `// <file>.go ...` comment naming what it implements.
`resume.go` starts at `package claudeengine`, and `resume_test.go` has no header either.

**Fix:** Add a file-header comment to each, in the package's style and with semantic line breaks.

### [NIT] Stat failure reported as a missing transcript

**Location:** internal/shuttleengine/claudeengine/resume.go:47

**Issue:** Refusing on any stat error is correct and fail-closed, matching card 6's "exists; otherwise refuse".
A permission error, though, produces "no transcript for session …", which misstates the cause; only the wrapped error reveals it.

**Fix:** Keep the not-exist message for `os.ErrNotExist`, and word other errors as "cannot stat the transcript".

### [NIT] Prepare's doc comment lists only two validations

**Location:** internal/shuttleengine/claudeengine/claudeengine.go:62

**Issue:** `Prepare` now also rejects a bad permission mode and a malformed resume id before writing artifacts, but its doc comment still names only Effort and Model.

**Fix:** Name all four validations.

### [NIT] No test pins that the DEFER turn end alone never re-qualifies

**Location:** internal/orchengine/watcher_test.go:975

**Issue:** The code requires a fresh turn end, because `toIdle` forgets `w.newest`.
The test appends turn end `b` immediately after the DEFER tick, though, so the deferral gate and the newest-turn-end gate coincide.
Nothing fails if a refactor lets the DEFER turn end re-qualify.

**Fix:** Add a test in which no turn end follows the DEFER and no soft request is sent after `soft_idle_s` has passed.

## Focus verdicts

- **isBoxRule:** the shortcut can accept a non-rule line below the caret; see F1.
  A non-rule line above the box cannot affect the result, because the real top rule sits directly above the caret and is found first.
- **tickIdle after DEFER:** not a defect.
  `toIdle` sets `w.newest = nil`, so the DEFER turn end cannot qualify, and only a fresh turn end can re-arm the soft trigger.
  The `LastDeferral` gate is implied by the newest-turn-end gate while the watcher runs, and it matches card 9's gate list.
  The test gap is F7.
- **DEFER detection on replay and in the entering tick:** not a defect.
  On replay, `deferRead` is the restart tick's time, which only delays the next attempt, so the error is conservative.
  Events read in the tick that enters handoff-requested are read while the phase is idle, so they are skipped.
  `enter` and `toIdle` both reset `phaseEvents`, so no deferral carries into a later cycle.
- **Smoke background-shell re-launch:** the dropped assertion is justified.
  The orchengine doc records that a background task survives `/clear` and that a resumed session may find it still running, and the test comment says the same.
  The weak handoff check beside it is F2.
- **--adopt ordering:** correct.
  The exclusivity check sits before `MkdirAll`, the lock and `reedUp`.
  The live-strand refusal tests `action != StartRelaunch`, which covers both the attach-only and the spawn-watcher-only branch.
- **checkResumable:** refusing on a non-not-exist stat error is correct; only the message wording is off (F5).
  Keeping the last unreadable-entry warning matches card 6's single "warning naming the path", and the scan continues, so a live holder later in the directory still refuses.
- **Comment convention:** see F3 and F4.
  The doc.go sections, watcher.go, run.go and smoke_live_test.go follow semantic line breaks.
- **Per-card mechanical checks:** no references to `orchShuttleConfig` or `TestOrchShuttleConfig_ClearsOnlyAgentDeny` remain outside `_lyx`.
  Every caller of `buildLaunchCmd`, `buildResumeCmd`, `denyInstalls`, `buildDenyNotice` and `buildSettings`, and every `StartSession` implementer (`runnerSessionStarter`, `fakeStarter`, `starterFunc`), is updated.
  No session-id validator or resume seam existed before (`validateSessionID`, `SessionResumer`, `Run.ResumeWarning` and `SoftThreshold`/`SoftIdle` are new).
