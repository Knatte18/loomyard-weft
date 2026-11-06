No findings.

Checked: F1 (no stencil-prose assertion left in `seatengine`, `burlerengine`, `burlercli`, `shedadapters` tests), F4 (`lensreport.Returned` has one production caller, `internal/burlerengine/assist.go:238`; `reportedLine` has no reference left), F2 (`AssistEnding` and `AssistLensesMissing` written only in `burlerreview.go`, read only in tests; `internal/shedadapters/doc.go:247` does not contradict the clearing), F5 (no comment still says every assistant includes `seat-directive-assistant`).

Tool calls used (3 of 6): grep over `internal` and `docs` for the stale phrases; grep for the two usage keys and `lensreport.Returned`; a narrower grep of four packages after the first was truncated.
