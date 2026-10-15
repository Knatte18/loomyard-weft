# Card 5 — Prune the shed driver, tackler and Bouncer prompt tests

**Edit:**
- `internal/sheddriver#TestTacklerPrompt`
- `internal/loomcli#TestDriverPrompt_ParentNotifyRuleFollowsWatched`
- `internal/shedadapters#TestBouncer_JudgeCall_PromptReadsFactsNotArtifacts`

**Delete:**
- `cmd/lyx#TestDriverStencil_ShowsOneBareStepCall`

**Intent:**
Prune the shed-layer prompt tests by D1:

- `TestTacklerPrompt` loses its row "the prompt is shed-layer wording with the stop kinds, the reading order and a two-path write rule": the stop kinds, the reading-order phrase and the arbiter ban are static prose of `shed-template-tackler.md`, and the verdict and case-brief paths it also names are already checked by the "every marker value" row.
  Its doc comment drops "the shed-layer wording".
  Every other row, the parent and edit directive checks and the leftover-marker check stay.
- `TestDriverPrompt_ParentNotifyRuleFollowsWatched` keeps only the anchors that tell the Go-chosen notify fragment apart: the watched row wants "Never message the parent that the run stopped, halted or finished" and not "At every escalation", the unwatched row wants "At every escalation" and not "Never message the parent that the run stopped"; "`parent_notice`", "a question you cannot settle yourself" and "`friction: failed`" go, being prose shared by both fragments.
- `TestBouncer_JudgeCall_PromptReadsFactsNotArtifacts` drops "Never relabel a BLOCKING finding downward", prose of `bouncer-template-judge.md`, and keeps "A parse-error row for the latest round means `CONTINUE`", which `bouncerprompt.go`'s Go-built decision rule renders.
  Its `//testtiming:keep` reason stops naming "the two guard sentences": it names the one Go-built decision-rule sentence the test still pins.
- `TestDriverStencil_ShowsOneBareStepCall` in `cmd/lyx/driverstencil_test.go` is deleted: the fenced step call, the retired subshell ban and the two envelope key names are static prose of `shed-template-driver.md`, and the keys' Go definitions are unexported to `cmd/lyx`.
  `TestDriverStencil_IsRecipeBlind`, a scan of the driver stencil family against Go-declared recipe and row names, stays.

**ImpactSummary:** test-only across `internal/sheddriver`, `internal/loomcli`, `internal/shedadapters` and `cmd/lyx`; the deleted test has no caller, and no production code changes.
