# Card 5 — LOOM's decision notice fits creel's line cap

**Create:**
- `plan:internal/loomcli#decisionWayNoticeTooLong` -> `const decisionWayNoticeTooLong = "way forward: shorten the decision or rationale, put the detail in a file and name that file in the entry"`
- `plan:internal/loomcli#noticeFits` -> `func noticeFits(deps decisionDeps, line string) bool`

**Edit:**
- `internal/loomcli#decisionDeps`
- `internal/loomcli#decisionState.preRun`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#notifyLiveRuns`
- `internal/loomcli#decisionFake.deps`
- `internal/loomcli#TestDecisionVerb_Refusals`
- `internal/loomcli#TestDecisionVerb_NotifiesLiveAgents`

**Uses:**
- `plan:internal/creelengine#LineFits`
- `internal/loomcli#decisionNoticeLine`
- `internal/loomcli#postDecisionNotice`
- `internal/loomcli#newDecisionFake`

**Intent:**
D1: LOOM's decision notice is a program's post that cannot take the file way forward card 4's line cap names, so `decision add` keeps its notices inside the cap itself.

`postDecisionNotice` signs a notice as the caller's strand or as the hub's LOOM program, so a notice must fit under either name.
`decisionDeps` gains `programName string`, the hub's full LOOM program name as `creelcli.NewProgramClient` forms it, `<shortname>:LOOM`; `decisionState.preRun` sets it from the target's hub shortname through `agentname.FormatProgram`.
`noticeFits(deps, line)` reports whether `creelengine.LineFits` holds for line under `deps.strandName`, when set, and under `deps.programName`.

`decisionVerb` refuses, after it builds the entries it would add and before it begins an amendment, appends or commits anything, an entry whose own one-entry notice does not fit: for each entry it renders `decisionNoticeLine(deps.recordPath, []AddedDecision{d}, true)` and, when `noticeFits` fails, refuses with `the decision notice for "<heading>" would pass the line creel delivers in one hook field; ` + `decisionWayNoticeTooLong`.
The input that reaches it is an entry whose decision and rationale together run past about 8 KiB, which `decisionEntries` and every earlier check accept.
`notifyLiveRuns` renders each receiver's line as now; when it does not fit and the call adds several entries, it sends one one-entry notice per entry in entry order instead, each of which fits by the refusal above, and counts the receiver notified only when every one was posted, reporting the first failure as today.
With one entry, the refusal guarantees the line fits.

Tests:
`TestDecisionVerb_Refusals` gains a row whose entry's decision is 9,000 bytes: the verb refuses naming the way forward, and the fake's record, commit and notify calls show nothing recorded, committed or sent.
`TestDecisionVerb_NotifiesLiveAgents` gains a row of a `--from` batch of two entries whose joined inline notice passes the cap while each one-entry notice fits, for a live Merriam strand whose notice inlines every entry: the fake's notify log holds two lines for that strand, one per entry, each opening with `decisionNoticeOpening`, and the strand is reported notified; in a second run of that batch the fake's notify fails the second one-entry post, and the strand is not reported notified while the envelope's notice failures name it with that post's error.
`decisionFake.deps` sets `programName` to `hub:LOOM`.

**ImpactSummary:** one new refusal before anything is recorded and a split of oversized batch notices, all inside loomcli's decision verb; its callers and the notice opening are unchanged.
