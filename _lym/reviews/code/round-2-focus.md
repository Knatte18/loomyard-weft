---
round: 2
focus:
  - "Tests of the assistant stencils in internal/burlerengine/assist_test.go and internal/seatengine/prompt_test.go: settle whether any assertion pins static stencil prose rather than a marker set, a value Go fills, or a fragment Go chooses, as PATTERN-test-speed's Test economy requires."
  - "internal/shedadapters/burlerreview.go acceptedExit and recordRoundAttempt: settle whether the usage record's assist_ending and assist_lenses_missing keys are reset on every accepted attempt, including one that ran no assistant, so a record never names an assistant that did not run."
  - "internal/burlerengine/review.go concludeReview: settle whether a test would fail if keepAssistReports received no lens names for a fanned round, so the missing-lens record is pinned through Review's result."
  - "internal/lensreport/lensreport.go Returned: settle whether it reads the outcome key of the opening fenced yaml block alone and tolerates CRLF and trailing whitespace."
  - "Comments in internal/seatengine table.go, doc.go and engine.go about the assistant directive block and what Run removes before the chair starts: settle whether each is still true now that a fanned assistant includes its own directive block and subagent reports are cleared."
---

Each entry names where to look; a fresh reviewer judges the diff as it stands.
