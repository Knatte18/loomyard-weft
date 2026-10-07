# Fixer report — plan round 1

All five findings are fixed in the plan files; `00-overview.md` is untouched, and `approved:` stays `false`.

## Changed

- F1: `17-burler-review-parse-gate.md` adds `ReviewGateEntry` as the one builder of the review entry, appends it in `Engine.Run`, and has `probeLiveRound` attach with the told gate plus that entry.
  The card adds `internal/shedadapters/burler.go` and `burler_test.go`, extending the existing probe-gate test with the entry's presence and its failed-gate mapping.
- F2: `14-loom-start-never-attaches.md` adds `internal/loomengine/config.go`, `internal/loomengine/seed.go` and `tools/sandbox/SANDBOX-CORE-SUITE.md` to its edits, each reworded to the success-envelope behavior.
- F3: `15-orch-start-never-attaches.md` deletes `OwnsTmuxEnv`, `ClientSession` and `SwitchClientArgv` from `internal/reedengine/client.go` with their tests in `client_test.go`.
- F4: `10-compaction-boundary-turn-ends.md` compares the two texts with whitespace trimmed, records the equality as an assumption to check against a real transcript, states the fail-closed bound and adds a whitespace-difference row to its table.
- F5: `02-claude-input-box-reader.md` drops the constant-pinning test of `SubmitSettle`.

## Gate repair

The gate blocked on `handle-unreferenced`: Card 17 declared a `plan:` handle for `ReviewGateEntry` that no other card uses.
The card no longer declares it; `ReviewGateEntry` stays described in the card's prose as a helper both of the card's own call sites use.
The informational `create-new-unit` note on Card 18 needs no change.

## Deferred

Nothing.
