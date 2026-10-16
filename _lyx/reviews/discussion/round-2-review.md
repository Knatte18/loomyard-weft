---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: "_lyx/discussion/decision-record.md (D17, next-wake fallback; D16, last paragraph)"
    summary: "The fallback for a dropped landing notice compares the board against the landings the orch handled, a record nothing keeps, so after a refresh, compact or handoff the orch cannot tell a missed landing from a handled one."
    fix: "In D17, replace the comparison against the landings it handled with a durable baseline and name it, either the landed lines in the hub notice log that D16 already appends before the hold-back check, read against the time of the orch's last update-plugins.sh run, or the installed lyx build's revision against main's HEAD. Reword D16's newly done and newly ready sentence to name the same baseline. Change the last acceptance row so the claims test pins that baseline in the fallback sentence."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: "_lyx/discussion/decision-record.md (D11, and the log-level-hygiene Scope bullet)"
    summary: "D11 says every production Warn call is checked and that only a non-loop site recording a real failure stays Warn, but then exempts every non-loop Warn site from a verdict, so a non-loop Warn logging an ordinary state is promised a check that no plan list or acceptance row ever reaches."
    fix: "Make the audit's reach match its finish line. In the Scope bullet, replace an audit of every production logger.Warn site plus every log line in a polled loop with an audit of every Warn or Info-or-above log line in a loop or polled path. In D11, replace the first sentence with one saying the audit covers every Warn line and every Info-or-above line inside a loop or polled path, delete the sentence A site that sits in no loop and records a real failure stays Warn, and keep A Warn site in no loop keeps Warn and needs no per-site verdict as the stated bound, ending it with the clause that such a site is out of this audit."
---

### [MEDIUM] Dropped-landing fallback rests on an unrecorded baseline

**Location:** `_lyx/discussion/decision-record.md`, D17's next-wake fallback paragraph and D16's last paragraph.

**Issue:** The focus asks whether every path that drops a landing notice leaves the orch a defined route to learn of the landing and run the deploy.
`rowNotifier.send` in `internal/battenshed/rownotice.go` drops a line on four paths: unreadable orch state, no orch strand recorded, `noticeMaxAttempts` failed `Notify` attempts, and a cancelled context.
D16 names only the first two, but D17's fallback does not depend on the cause, so every path reaches it.
The fallback itself is not defined, though.
D17 has the orch compare the board's done tasks against the landings it handled, and D16 says the orch checks the board for tasks newly done and newly ready.
Neither names where the handled landings, or the baseline for newly, are recorded.
The orch's context is the only carrier, and `ly:refresh`, `ly:distill` and a handoff reset it, so after one of them the orch either reruns `update-plugins.sh` for every done task or misses the dropped one.
A durable record already exists: D16 appends every landed line to the hub notice log before the hold-back check (`appendNoticeLog` runs first in `send`).
The fallback should key on that record or on another durable one.
The current orch stencil (`contracts/stencils/orch/orch-template-role.md:130`) already says "at your next wake after any landing" with the same undefined baseline, so this decision is where it gets fixed.
The undefined baseline came in with round 1's F3 fix, which applied that review's proposed wording verbatim.
This finding is a gap in that wording, not a regression, and the fixer report disputed nothing.

### [MEDIUM] D11 promises a check on every Warn site that its finish line exempts

**Location:** `_lyx/discussion/decision-record.md`, D11, and the `log-level-hygiene` Scope bullet.

**Issue:** The focus asks whether D11's finish line is bounded and checkable against built code for both Warn and polled Info-or-above lines.
The finish line is bounded: the plan lists only loop and polled sites with a verdict, and the acceptance row checks those listed lines.
D11's other sentences say more.
"Every production `logger.Warn` call is checked against the policy" and "A site that sits in no loop and records a real failure stays Warn" imply that a non-loop Warn logging an ordinary state is demoted.
"A Warn site in no loop keeps Warn by the rule above and needs no per-site verdict" then keeps every non-loop Warn without a check.
The production tree holds several hundred `logger.Warn` calls (`grep -rn "logger\.Warn(" --include=*.go internal cmd tools`), and none of the non-loop ones reach a plan list or an acceptance row.
The plan writer gets two readings, and the one that checks every site has no finish line.
The Scope bullet repeats the broad claim.
The fix narrows the stated reach to what the finish line covers.
The targeted non-loop sites are already decided in D5, D6, D7 and D10.
Round 1's F2 fix added the bounded finish line but left D11's opening sentence and the Scope bullet unchanged, which is where the two readings come from.

## Rejected

- Focus 1, D13 test-build import cycle: not a defect.
  `internal/logger/sink.go` already imports `testing` and calls `testing.Testing()` for the durable half.
  Switching the stderr half's default in the same package adds no import to any package, so no test build in `internal/logger`'s transitive imports (`dotgit`, `lyxcwd`, `lyxdirs`, `proc`) gains an edge.
  `configureFromEnv` applies `LYX_LOG_FILE` through `SetOutput` after the default, so an operator-set log file still wins under `go test`.
  The one test that reads a real `os.Pipe` (`claudeengine/envfile_integration_test.go`) does not read logger output.
- Focus 4, D5 wait-report dedupe: not a defect.
  D5 has the wait report write the reason file whenever the text changes and log only when the key changes, so the file holds the newest text while the key is unchanged.
  It names the key's source: each caller passes it beside the text, instead of the key being parsed out of the formatted reason.
  The acceptance row pins the newest-text property.
  This matches `innerRunProducer.report`'s current shape, which gates `reportStuck` on a text change.
- D14's named accessors exist: `fabricengine.HubLogsDir` (`internal/fabricengine/junctionnames.go:175`) and `verifytree.Dir` (`internal/verifytree/verify.go:157`).
  The composite's `Topology.Remove` call is in `internal/pairteardown/teardown.go:132`.
