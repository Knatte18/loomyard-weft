# Plan — round 1 fixer report

## Changes

- **F1** (BLOCKING), fixed as proposed.
  Card 1 lists `internal/websterengine_test#newRunFixtureOver`, `internal/websterengine_test#newRecoverFixtureOver` and `internal/webstercli#newVerbsFixture` under `Edit:`.
  Its Intent says these fixtures drop `Engine` `claude` from their role literals in the same commit, leaving the empty engine their `shuttlefake.EngineSet` defaults to.
  Card 7 keeps all three targets for its move to `shuttleengine.ModelChoice`, and its test bullet now says the fixtures keep card 1's empty engine, no longer that card 7 is the card that removes `claude`.
- **F2** (MEDIUM), fixed as proposed.
  Card 10 no longer lists `TestRun_RefusesBeforeSpawn` and `TestRun_AutoRebaseline`, and their planned rows are gone.
  The `master engine not registered` refusal-spec row drops `run`'s partition formation and auto-rebaseline as triggers.
  A new Intent sentence says `Run` still calls the lister but card 9's `roleAuditEngine` gate has already refused the input, so the lister's error reaches `Run` only as an internal error.
- **F3** (MEDIUM), fixed as proposed, using the `ModelKey` option.
  `ModelKey` gains `Entry`.
  `Wrap` gets a fourth form for a list entry: `set the model of <Entry> in <Key> in <Module>.yaml ... with lyx config <Module>`.
  Each advisor carries `ModelKey{loom, discussion_advisors, Entry: <advisor name>}`, and the advisor's refusal-spec row names the advisor's entry.
  The `TestRunner_Requires` and advisor-test rows assert this text.
- **F4** (MEDIUM), fixed as proposed.
  `checkRequirementsAtStart` keeps today's generic way forward.
  It returns the refusal as a new unexported `startRequirementsError`, added under `Create:`, which holds the refusal without the way forward and still unwraps to `ErrRequirementsUnmet`.
  `Wrap` puts the key-specific way forward in place of the generic one only when the error's chain holds that type.
  So `lyx orch start`'s unwrapped refusal keeps its way forward.
  Unknown-requirement and attach refusals pass through unchanged.
  `TestRunner_Requires` asserts that an unwrapped refusal carries the generic clause and that each wrapped form carries only its own.
  The Intent's "no refusal without a way forward" sentence now covers every `Start` caller, and the ImpactSummary is updated to match.
- **F5** (MEDIUM), fixed differently in part.
  The `StartFailure` table's two-seat chair row now has the zero `ModelKey`, as on a MultiLLM seat, and asserts the zero key's seat way forward and no `AddStrand`, as proposed.
  For the advisor row, I did not make the proposed change: starting the chair on the same real `Runner` over `shuttlefake.EngineSet(&shuttlefake.Engine{})`.
  Card 11's `seatSpec` makes every seat of a multi-seat table require `peer-messaging`, chair included.
  `shuttlefake.Engine` implements no `RequirementSupporter` (`internal/shuttleengine/requires.go`, `meetsAll`), so a chair on that set is always refused before any advisor starts.
  Instead, `fakeShuttle` gains a field that maps a role to the `Shuttle` that starts it, and the advisor's start goes through `RunnerShuttle` on that real `Runner` while the chair stays on the fake handle.
  Shuttle's real refusal therefore runs for the advisor.
  `fakeShuttle` and `fakeShuttle.StartGated` are added to card 11's `Edit:` list, and the Intent says why the chair cannot share the set.

Also in card 11: the `TestDarnSpec` bullet had been run onto the previous line with no line break, and now has its own line.

`lyx loom validate-plan --lens-review` passes after these edits, with only the existing informational `create-new-unit` finding on card 13.

## Disputed

None.

## Deferred

None.
