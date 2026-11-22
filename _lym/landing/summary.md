# Launch ignores unused gate keys; add `lym config --unset`

A hub `gate.yaml` that held the retired `gates.harden-live-copies` entry with no `scope` stopped `lym s` from resuming the orch session.
`claudeengine` loaded the whole gate config only to read its `refuse_raw` list, so one bad gate key refused the launch.
`lym config` had no verb to remove a key, so only the interactive editor could clear it.

## What changes

- A session launch reads only the gate config's `refuse_raw` list, through the new `gateslot.LoadRefuseRaw`.
  When the full load fails past the layer merge, the launch logs a warning that names the way forward and installs the list read from the merged stack.
  Gate calls still refuse until the config is fixed.
  `claudeengine` and `codexengine` both use it.
- A stack that does not merge, or an invalid `refuse_raw` list, still refuses the launch.
- `lym config <module> --unset <key>` removes a key or a map entry with no editor, on any layer.
  It is repeatable and is mutually exclusive with `--set`, `--print` and `--resolved`.
  A mapping the removal empties is removed with its key.
  An absent file, or a key the file does not hold, is an error and writes nothing.
  A result the module's loader newly refuses leaves the file as it was.
  The implementation is `yamlengine.RemoveKeys` and `configengine.Unset`, and `configcli` routes both `--set` and `--unset` through one `layerWrite` path.
- `gateslot.RetireHardenLiveCopies` now also removes a `harden-live-copies` entry the gate config's own entry check refuses, a scope-less one included.
  When its tiers differ from `harden-live`'s single tier, the removal is flagged with the way forward for `harden-live`.
  An entry that loads and differs is still left in place and flagged, now naming `--unset` as the removal.
- Docs follow: `docs/overview.md`, `docs/shared-libs/configengine.md`, `PATTERN-config-layers.md` and the package docs.

## Design decisions

- `--unset` removes a key the template lacks.
  This makes it a second operator route for removing such keys beside `lym config reconcile --apply`;
  the hubreconcile walk still removes none on its own.
- The launch fallback reads `refuse_raw` from the merged stack rather than skipping the gate config.
  The shell guard therefore stays installed whatever else is wrong in the file.

## Check by hand

- `lym s` resumes an orch on a hub whose `gate.yaml` holds a scope-less `gates.harden-live-copies`, and logs the warning.
  This was not run against a real hub.
- `lym config gate --unset gates.harden-live-copies` from the prime on such a hub, then a clean `lym config gate --resolved`.
- `RetireHardenLiveCopies` still runs under the existing `RetiredCopiesReinterpretationID` (`gate-harden-live-copies-retired`).
  The decision record chose a new ID so hubs past the first marker run the widened rewrite once more,
  but the diff registers none: check whether a hub already marked keeps a scope-less entry until `--unset` is run by hand.
- Open review findings:
  - `internal/hubreconcile/doc.go` names reconcile as the one removal, and now also names `--unset`; check the wording against the other docs that describe key removal.
  - A hub already marked by the first reinterpretation never gets the scope-less removal (the ID point above).
  - The `refuse_raw` fallback in `gateslot.LoadRefuseRaw` has no test of its own.
  - `--unset` removes a key by hand, which sits against the rule that only reconcile removes a key the template lacks; check that the `configcli` and `hubreconcile` docs state the exception consistently.

## Merge danger

**Door:** one-way
**Blast radius:** gateslot
