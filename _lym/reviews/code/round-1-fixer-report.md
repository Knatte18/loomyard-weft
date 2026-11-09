# Fixer-report, round 1 (code)

## Changes

All four findings fixed as proposed; each is its own commit.

- F1 (`fbc13466`): `TestDispatchSetProfiles` gained two rows.
  `light.loom.loom.plan_max_bounces=2` wants the hub file with that one value changed.
  `light.loom.loom.no_such_key=1` wants code 1, the file as seeded and `run profile "light"` in the error.
- F2 (`7f83eaa1`): `TestSetEntries` gained rows for a bracketed value, a malformed flow value, `true` and `null` tags,
  a null parent and a replaced leaf's line comment.
  The `true` row wants `on: true` unquoted; the first run showed the output leaves the key `on` unquoted, so the row was corrected to that.
- F3 (`88ec7cd2`): the `Spec.JSONEntries` comment in `internal/configengine/layers.go` names the undotted and dotted forms.
- F4 (`b2613f4f`): one sentence in `internal/configcli/doc.go` and in the `SetEntries` doc comment says a dot in a key always separates levels.
- Self-check follow-up: the `ConfigSpec` comment in `internal/runprofile/runprofile.go` now names the dotted form beside the whole-entry JSON object.
- Gate repair: that comment first wrapped mid-sentence; it is now one sentence per line.

`lym gate test ./internal/yamlengine ./internal/configcli ./internal/configengine` passed: 707 passed, 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, `internal/runprofile/runprofile.go:38` comment stale beside the dotted form; fixed as proposed.

No fork returned without a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
