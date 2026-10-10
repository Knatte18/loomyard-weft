# Card 3 — rebaseline-refusals-keyed

**Edit:**
- `internal/websterengine#RebaselineDeps`
- `internal/websterengine#Rebaseline`
- `internal/websterengine#rebaselineBatches`
- `internal/websterengine#recordedOverviewFrame` -> `func recordedOverviewFrame(st *State, websterDir string) (frame, cause, path string, err error)`
- `internal/websterengine#overviewRefusal` -> `func overviewRefusal(recorded, cause, path, step string) error`
- `internal/websterengine#fingerprintMismatchWayForward`
- `internal/websterengine/rebaseline_test.go`
- `//contracts/specs/refusal-spec.md`

**Create:**
- `plan:internal/websterengine#followUpLanding` -> `func followUpLanding(step string) string`

**Intent:** Key every clause of a rebaseline refusal on the step the caller names, and name the cause of an empty overview frame (D1 and D3).
`RebaselineDeps` gains `Step`, the step its refusals name: empty means `lyx webster run`, as `RunDeps.ReentryStep` does, and the loom's Webster row sets its re-step text (card 4).
Every clause that names `lyx webster rebaseline` today is keyed the same way: the follow-up-card landing, the changed-but-not-named card set, and each transient retry.
With an empty `Step` the text is today's; with a step set, the landing says to fix the plan, move a done card's edit to a follow-up card after the last begun batch with its Card Index line in `00-overview.md`, and re-step the row, the unnamed-card clause says to re-step the row after fixing the plan, and a transient says to re-step the row, never to run `lyx webster rebaseline` by hand.
`followUpLanding` renders the landing for a step; the `followUpCardLanding` constant stays as the manual text `fingerprintMismatchWayForward` and `followUpLanding("")` return.
`recordedOverviewFrame` returns the cause of an empty frame beside it, one of three fixed texts, no overview hash recorded, the baseline copy absent, or a copy without a parseable Card Index, and the baseline copy's path where there is one; the helper logs nothing.
`Rebaseline` logs the cause at Info at its own call, with the path when there is one, so a rebaseline logs it once; `fingerprintMismatchWayForward`, its other caller, drops the new returns and keeps its no-frame text unchanged.
`overviewRefusal`'s no-frame text names the cause and the path, and the refusal-spec row `rebaseline overview changed` carries the new wording; the other rebaseline rows carry the step-keyed alternative beside today's text.
`TestRebaseline_OverviewFrameUnreadable` and `TestRebaseline_EditedPlan` gain the rows the record's acceptance names: once each for no recorded overview hash, an absent baseline copy and a copy without a Card Index, the last two naming the path, with the log line captured; `TestRebaseline_CardSet` gains a step-set row per keyed clause asserting no text contains `lyx webster rebaseline`.

**ImpactSummary:** Two websterengine helpers re-signed with their two callers in the same package; the manual refusal texts and the CLI's rebaseline verb are unchanged.
