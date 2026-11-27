```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to the board entry's goal points.

**Decisions** (the board entry's `## Goal` in `_lym/discussion/decision-record.md`; the record has no `Added after Discussion` entries):
- G1: find which harness setting backgrounds a foreground Bash call at 180 s.
- G2: either set it in the launch and resume lines, or keep the keep-warm cap below it.
- G3: so a gate call always answers while still in the foreground.

**Changes, and what each traces to:**
- C1. New constants `subagentPromptCacheTTLEnvKey` and `subagentPromptCacheTTL`, and their assignments in `buildLaunchCmd` and `buildResumeCmd` (`internal/shuttleengine/claudeengine/command.go:101-111,375,619`). Traces to G2, the first option. The constants' doc comments record G1's answer.
- C2. `OpenLaunch` exports the same variable ahead of the fresh and resume open lines (`command.go:429-430,523,526-527`). Traces to G3. The entry names "launch and resume lines", but G3 needs every session that can fork, and the orch and other sessions opened with `lym shuttle open` fork too. This is the goal reached on one more launch path, not unrequested scope.
- C3. The test env leads, the `TestBuildLaunchCmd` and `TestBuildResumeCmd` one-hour rows, and the `openLines` lead (`command_test.go:23-30,93,386,872`). Forced by C1 and C2.
- C4. The prose in `doc.go:88-92`, `template.yaml:8` and `docs/overview.md:717`. Forced by C1: the old text "forks run at Claude Code's 5-minute subagent lifetime; a cap above five minutes leaves forks' caches cold" became false, and the claude engine's docs describe its launch-line env.
- C5. Commit 8c3a7047b re-breaks the comment lines that C1 and C4 added. Forced, a format fix of lines C1 and C4 introduced.

**Decisions, and the changes that build each:**
- G1: built by the cause stated in commit 34fc72cb5's `Root-Cause:` trailer and summary, and recorded in C1's doc comments and C4. No variable sets the 180 s itself: Claude Code exempts a subagent whose prompt cache is one hour. The bundle at `Nws` (`iGi=180000`, exempt when `k==="1h"`) bears this out. No diff hunk is owed beyond recording it.
- G2: built by C1, taking the "set it in the launch and resume lines" option.
- G3: built by C1 and C2 together, which cover every line a claude session is started, resumed or opened by.

**Findings:** none. Every change traces to a goal point or is forced by one that does, and every goal point has a change that builds it.

**Tool calls:** none; my inherited context held everything this lens needed.
