---
round: 1
focus:
  - "internal/slackcli/cli.go preRun: settle whether `args[0]` and the toggle flags are always set when a toggle's pre-run fails (cobra's order of arg validation, required-flag checks and PersistentPreRun), and whether the `cmd.Name() == toggleVerb` test can match another verb or miss the toggle."
  - "internal/slackcli/toggle.go reportToggle: settle whether the Warn reaches the trace log in the pre-run path, where the logger may not be initialised yet, and whether any path reports a failed toggle twice or not at all."
  - "internal/slackcli/wiring.go and cli.go: settle whether moving `showToggle` into `productionCLI` leaves every other construction of `slackCLI`, tests included, with a nil `showToggle` that `reportToggle` would call."
  - "internal/reedengine/slacktoggle.go: settle whether tmux's `display-message -d` takes milliseconds as the constant's comment says, and whether the ten-second hold also applies to a success message and is wanted there."
  - "The decision record's `## Goal` (issue #212): settle whether the diff answers both halves, a failed click shown on the bar and a trace-log entry, and whether it touches the report's other cause, the `@lym_orch_dir` and `@lym_button_dir` lookup, or the click line that discards output."
  - "Commit message trailers: settle whether `Root-Cause:` names a defect the diff removes."
  - "Tests: settle whether `ARefusalBeforeTheToggleStartsIsShownOnTheBar` asserts the Warn log entry the diff adds, and whether `wireProduction`'s unreached `showToggle` assignment change has a check that a production click reaches `ShowToggleOutcome`."
---

The diff is small: a pre-run refusal of `slack toggle` now redraws the clicked bar, every failed toggle is logged at Warn, and the message holds for ten seconds.
