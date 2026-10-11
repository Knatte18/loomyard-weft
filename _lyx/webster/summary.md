# shuttle-hook-first: hook-first session state for shuttle

Both batches reported `status: done`.

## Batch 01-hook-recording-capture

Head `5da4b3fa73cbd222f91b103db319c874fda13fb1`.
Cards 1–5 landed: `SessionStart` for every source and `PermissionRequest` are recorded on every run, and a real hook corpus is captured under `claudeengine/testdata/corpus/hook-capture/`.
The provider-neutral session-start source type maps Claude's source strings, and the session fold has an idle-fresh state a session start sets, read as ready and as idle.
`PermissionRequest` reads as an ask, every `Notification` type is mapped, and an interrupt releases an ask.
A fork transcript's pending call is read from its newest tool-calling message only.

Reported deviation from the declared file-ops: `internal/shuttlecli/modelpin_test.go`, whose real-claude spawn-site count rose with the new capture scenario.
Per the fork, the corpus shows `SessionStart(startup)` landing on its own before the first prompt, so card 7 was not blocked.

## Batch 06-clear-compact-confirm

Head `03928082e4d0b7d16fa8ae07246467f7d24756f9`.
Cards 6–11 landed: `/clear` and `/compact` confirm on their matching `SessionStart` line past the pre-send offset, and a run starts only once a `SessionStart` line and the ready pane both hold.
A registry reading or a process-ending `SessionEnd` ends a run `died`, or `done` when every output file exists.
The `StrayInputReader` capability and Claude's lone stray-character judgment were added, a lone stray character is cleared once behind a full guard reading and then read as residue, and batten's driver-busy notice carries the refused resume's box text.

No deviations were reported for this batch; the digest carried only informational scope warnings.
