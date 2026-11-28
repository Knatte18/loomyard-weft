Lens fix-affected-set: I reran the searches behind the set-based findings (F1, F4, F6, F12) and checked that the fix covers every member each search finds.

**Findings left unchecked:** F5, F7 and F11. Their searches are the ones the fixer ran in this round, and my tool-call cap went to F4 and F6. F3, F8, F9, F10, F13 and F14 have no set premise.

- **MEDIUM — card 4, the rewritten `refuseUnsuppliedSeams` sentence ("The handover readers `OriginParentRunStatus` and `OriginBattenStatus` meet it only through `originStatusBytesAtTip`… left to part 4")**
  - The F4 set of readers that would meet the refusal mid-flow is wider than this sentence and the two push verbs.
  - `ReadUpstreamTakeover` also has a production caller outside the push verbs: `internal/loomcli/landingdeps.go:878`, `runHandOver`, which is the landing hand-over seam.
  - `internal/fabricengine/takeover.go:171` also calls `primeTakeovers`, which reads `ReadUpstreamTakeover` (`takeover.go:137`). I did not read the function that encloses line 171, so I cannot say whether a push verb covers it.
  - These callers reach the refusal through `originStatusAtTip`, which goes through `originStatusBytesAtTip`, after they have started acting. The sentence names them neither as covered nor as left to part 4.
  - Evidence: the `ReadUpstreamTakeover(` grep hits `takeover.go:112`, `takeover.go:137` and `loomcli/landingdeps.go:878`. Only the `takeover.go:112` hit, `takeoverStop` inside `retryWeftPush`, sits in a push path.

- **LOW — card 12, the new fixtures bullet and Create lines for the moved push rows**
  - The helper set lists what the moved rows call directly, but not what those helpers call.
  - `newPairFixture` calls `newFailureFixtureAt` (`pushclasses_integration_test.go:133-135`).
  - The failed-push assertions read through `failedPushesOf` (`pushclasses_integration_test.go:59`).
  - Today's records fixture also runs `registerHeddleReplayer()` beside `boardkit.InstallReplay` (`replay_integration_test.go:37-40`, `47-50`). The new bullet says `failureFixture` installs the replay through `boardkit.InstallReplay` alone.
  - Unverified: whether `boardkit.InstallReplay` already registers heddle's Bolt replayer. I did not read it.

- **Covered:**
  - F12: `transport failure or failed` appears only at `internal/fabricengine/doc.go:652`, and none of the deleted function names appear in either doc.go or in `pattern/`.
  - F1: no other plan card places a test in package `fabricengine`.
  - F6: both `OwedUnrecorded_*OfAnEndedEntry*` rows sit in `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`, and their helpers are all listed.

Calls made:
1. Grepped the `ReadUpstreamTakeover`, `originStatusAtTip`, `OriginParentRunStatus` and `OriginBattenStatus` callers, the plan cards for package-`fabricengine` tests, doc.go and `pattern/` for the class list and the deleted names, and the helpers of the `OwedUnrecorded` ended-entry rows. My context held only the fixer's narrower greps.
2. Found the functions that enclose `takeover.go:112` and `:137` and `loomcli/landingdeps.go:878`, to tell push-path callers from other callers.
3. Grepped the `primeTakeovers` callers and read `runHandOver`'s body, to confirm the hand-over caller acts before it reads the takeover.
