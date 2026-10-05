# Round 2 fixer report

Every finding is fixed; nothing was deferred.

## Changed

- F1, F2, F5 (`_lyx/plan/20-drop-getter-delegation-tests.md`):
  the `CompactedSince` bullet now says the claudeengine tests cover the detection on the engine directly, the refusal test covers the missing capability, and the Runner's pass-through keeps no test by choice;
  it also says why `TestRunner_ContextTokens_Delegates` stays.
  The replacement test now asserts a live-strand `SkillUnknown` true and false case.
  `fakes_test.go` left the Edit list, and the `ImpactSummary` names only `session_test.go`'s fake state.
- F3, F4 (`_lyx/plan/21-skill-skip-tables.md`):
  the startup row shape is stated (`spec`, `endsTurn`, `unknown`, `wantTyped`, `minElapsed`, `maxElapsed`).
  The watcher table gains a third optional step, `endTurns`, and each row's final assertion is named, so the card no longer contradicts its own limit.
- F6, F7 (`_lyx/plan/17-orch-role-run-loop.md`):
  the After landing bullet states that the friction-notes-and-report step stays, read from the prime's own durable directories after the pull.
  The claims paragraph adds must claims for the reject form, for Finalize marking the board task done and pushing, and for the post-landing reading, and a mustNot claim for "start the run again".

## Deferred

None.
