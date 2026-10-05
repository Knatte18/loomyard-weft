# Round 1 fixer report

## Changed

- F1 (MEDIUM): `_lyx/plan/01-claude-env-file.md` Tests gained a bullet updating the tail assertion of `TestBuildSettings_DenyToggleMatrix`, which pinned one PreToolUse entry (the removed Bash stdin rewrite), to expect none.
- F2 (MEDIUM): `_lyx/plan/03-driver-bare-step.md` now has each of `## Error envelopes`, `## Parking` and `## Repair records and the stop report` state the `envelope_path` source in a sentence of its own, with no pointing back.
- F3 (LOW): `_lyx/plan/02-short-step-envelope.md` gained `docs/overview.md` in `Edit`, plus a paragraph correcting the one stale sentence on the step envelope's keys.
- F4 (NIT): `_lyx/plan/02-short-step-envelope.md` now extends the new table with the `last_step.envelope_path` read and leaves `TestStatusCmd_LastStep` unchanged.

`00-overview.md` is untouched, and `approved:` was not written.

## Deferred

Nothing.

## Note

F3 adds `docs/overview.md` to card 2, which the decision record's D4 says stays unchanged.
The record's premise (no module or execution-stack change) holds, but the overview sentence on the step envelope's keys describes the changed CLI behavior, and CLAUDE.md requires such docs to land in the same commit.
The record is the answer key and was not edited; the orchestrator may drop the edit if D4 is meant literally.
