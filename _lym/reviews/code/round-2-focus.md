---
round: 2
focus:
  - "internal/boardengine/replay.go (the beam_color row in replayFields) and internal/boardengine/task.go (normalizeTask dropping beam_color from a non-beam entry): does a test in internal/boardengine/replay_test.go or the normalizeTask/round-trip tests fail if the row or the drop is removed?"
  - "internal/fabricengine/hubworkspace.go regenerateHubWorkspace and beamWorkspaceSettings: when one beam's anchor record or .vscode/settings.json is unreadable, do the remaining beams' files and removeStaleBeamWorkspaces still run, and is the case tested in hubworkspace_integration_test.go?"
  - "internal/fabricengine/doc.go section 'The hub workspace file' and internal/beamcli/doc.go create step 2: do the docs state that a beam's workspace file splices the beam's own settings and that SetOwners records the beam colour and re-runs for a beam recorded here with no colour?"
  - "internal/boardengine/store.go UpsertPayloadKeys and the lym board upsert payload help in internal/boardcli/cli.go: is the status of beam_color in a lym board get payload (output-only or accepted) stated and covered by an upsert key test row?"
  - "internal/ideengine/spawn.go lookFor and internal/beamcli/create.go SetOwners call: are the unreadable-board and failing-ParentOf fallbacks, and the BeamColor passed by lym beam create, each pinned by a test row that fails when the behavior is dropped?"
---

Each entry restates its sites so the reviewer needs no earlier round.
