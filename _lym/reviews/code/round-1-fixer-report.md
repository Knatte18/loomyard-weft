# Fixer report — code round 1

## Changes

- **F1** fixed as proposed: `internal/fabricengine/doc.go` now says the hub file splices the prime's settings and a beam's file the beam's own when it exists and can be read (commits 48a277c2, cecd164e).
- **F2** fixed as proposed, with a limit: `replay_test.go` gains a replay row for a beam colour the commit sets (it fails without the `replayFields` row) and a row for a cleared beam flag.
  `store_test.go` gains a beam_color row to the load/save drop test.
  The `normalizeTask` drop at `task.go:279` is not pinned by any row: with that line removed every test still passes, because `migrateEntries` drops the colour first on every path that parses a board.
  The rows pin the observable behaviour; the line itself is shadowed.
- **F3** fixed as proposed: `regenerateHubWorkspace` logs a Warn naming the beam and splices the prime's settings when `beamWorkspaceSettings` fails, so the remaining beams and the stale-file removal still run.
  `hubworkspace_integration_test.go` gains an unreadable-settings step in the beam settings test.
  The `RegenerateHubWorkspace` comment and the module doc state the fallback.
- **F4** fixed as proposed, first option: `UpsertPayloadKeys` and the `internal/boardcli/doc.go` package doc state that `beam_color` is output-only; `store_test.go` gains a row asserting an upsert naming it is refused.
- **F5** fixed as proposed: `TestLookForDegradesToATaskUnderThePrime` covers an unreadable board and an unusable board-recorded anchor, so `parentOf` fails.
  The two cases are rows of one new table test because they need a damaged hub each, which `TestLookFor`'s shared hub cannot give.
- **F7** fixed as proposed: `internal/beamcli/doc.go` step 2 names the colour and the re-run.

## Self-check

- fix-code-correctness, MEDIUM: the F2 load/save row does not pin `normalizeTask`'s drop. Fixed with my own change: added the cleared-beam-flag replay row, which reaches `normalizeTask` through `replay.go:392`. A mutation run showed that row too passes without the `task.go:279` line, so the line stays unpinned; recorded under F2.
- fix-code-correctness, LOW: `UnresolvableParent` leaves nothing behind. Not a defect; no change.
- fix-consistency, LOW: module doc stale against the F3 fallback. Fixed as proposed.
- fix-consistency, LOW: `beamWorkspaceSettings` comment described its caller. Fixed as proposed, the sentence restored.
- fix-proposal, LOW: F2 row does not cover `normalizeTask`. Duplicate of the first fix-code-correctness finding.
- fix-proposal, LOW: F4 output-only wording leaves the read-edit-upsert round trip refused. Not changed: the review offered both options, D1 says `beam_color` is no upsert key, and refusing is the documented behaviour; the help and comment now say so.
- fix-proposal, LOW: F1 sentence stale against F3. Duplicate of the fix-consistency finding.
- fix-affected-set, LOW: `RegenerateHubWorkspace` comment still prime-only for beam files. Fixed as proposed.
- fix-affected-set, LOW: the same comment omits the F3 fallback. Fixed as proposed.

## Disputed

- **F6**, premise false: the review says the fake `SetOwners` ignores `BeamColor`.
  In `internal/beamcli/create_test.go` the fake returns `errors.New("the beam owner write carries no colour picker")` when `req.BeamColor == nil`, so dropping the field in `create.go:164` fails every create row that reaches the owner step.

## Deferred

Nothing deferred.

Tests run: `lym gate test --tier 0` over `internal/boardengine`, `internal/boardcli`, `internal/beamcli`, `internal/ideengine`, and `--tier 1` over `internal/fabricengine`, all green before the self-check edits; `internal/boardengine` tier 0 re-run green after them.
The `internal/fabricengine` comment-only edits after the self-check were not re-run at tier 1.

## Gate repair

The verify gate failed tier 0 on two findings of this round's own changes:

- `TestCLIForm_EveryCommand`: the `lym board upsert` help grew to 2096 bytes against a cap of 2000.
  The `beam_color` line left the help text for the `internal/boardcli/doc.go` package doc, as the test directs; the `UpsertPayloadKeys` comment keeps the statement.
- `TestEnforcement_FabricVocabulary`: `internal/ideengine/look_test.go` imported `internal/weftname` outside its owner set.
  The unresolvable-parent row now damages the board's recorded anchor through `fabricengine.BoardDir` and `lymcwd.AnchorFileName`, and builds its `own` location before the damage.

`lym gate test --tier 0` over `cmd/lym`, `internal/lymcwd`, `internal/ideengine` and `internal/boardcli` is green.
