---
round: 1
focus:
  - "Verify gate spans against 'as the Webster rows do' (internal/weaverengine/verify.go timedVerify, run.go finishRun): trace how Webster builds its verify entry's spans, and settle which span kinds it records, including time spent held waiting for a gate slot (deps.Geom.VerifyHold, shuttleengine.GateSpanHeld), and whether Weaver's gate and rework spans measure the same intervals."
  - "Paths on which a call runs or waits on the plan-level verify or a verify fixer (resume settle of a left-running fixer, re-entry after a verify stop, stuck, error and cancellation returns through run.go and internal/shedadapters/weaver.go): which spans reach the row's history entry, and does any path lose or misstate one? Check the change description's statements about this against the code."
  - "Attempt timeout (internal/weaverengine/strand.go, run.go runBatch, resume.go resumeAttempt): from which instant the deadline is counted on spawn and on attach, per internal/shuttleengine, and whether config.go, template.yaml, doc.go and the attemptLaunch comment all state that behaviour correctly. Check that every site that waits on an attempt strand sets the bound."
  - "summary.md deviations and departures (verify.go writeSummary): is every done batch's digest populated on every route to done (normal end, resume, re-entry record-done-without-spawn), and does the rendering stay well-formed and comparable with what Webster's summary names, for multiple or unusual trailer texts and file lists?"
  - "Tests added for the four paths (run_test.go, resume_test.go, run_fixture_test.go, shedadapters/weaver_test.go): does each one drive the path its name states (paneDies against the lost-strand classification, the third rung, the resume rows, the failed and exhausted verify), assert from independent values, and avoid duplicating an existing test? Is any behaviour the diff adds left uncovered, such as span times or the attach bound?"
  - "Documents the repository names for this kind of change (PATTERN.md and the pattern/ files, contracts/specs, docs/overview.md, package docs): does a new weaver.yaml key, a new summary content or the new row record require a doc edit that the diff lacks?"
---

Seed focus for round 1 over `git diff 1ae03e17f62aa83fbd81bfa9dbadabad3645366e..HEAD`.
The answer key is the decision record's board entry, which asks for four changes to Weaver.
The entries are questions to settle, not verdicts.
