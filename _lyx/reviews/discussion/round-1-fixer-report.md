# Fixer report — discussion, round 1

All edits are in `_lyx/discussion/decision-record.md`; `support-log.md` needed no change.

## Changes

- **F1** (fixed as proposed, first option): D3's Decision defines the created directories as the entries `CardTargetDirs` yields with `DeleteOnly` false for each card at or below the Verify's card (the whole plan for the overview chain), a directory counting when it is one of them or an ancestor of one.
  `CardTargetDirs` (`internal/planparser/targetdirs.go`) walks every group, Custom included, and maps member glyphs through `UnitPath`, so both gaps close.
  Acceptance gains the member-glyph Create and Custom rows, and the Notes bullet pointing at the `path-missing` unions is cut down to the `verifymodule.go` helpers.
- **F2** (fixed as proposed): Scope In names refusal-spec's shed `step interrupted` row.
  D5 gains a bullet saying that row names both causes, leaves the status file as found and gives D5's way forward.
  The `PATTERN-refusal-way-forward` Constraints bullet names the row.
- **F3** (fixed as proposed): D5's way forward is split by cause: the recipe's reseed at `status-missing`, the `--until-stop` re-step at `status-unreadable`.
  D5's Bound adds that a file that later reads `running` is resumed by that re-step, not `lyx loom resume`, and that `lyx loom arbiter` refuses either stop under its `status or envelope unreadable` row.
- **F4** (fixed as proposed, first option): `status-unreadable` with no re-step applies to a decode error only.
  A read-lock or raw read error re-reads once under the loop's existing single re-step (`restepped` in `runLoop`) and stops as `status-unreadable` only if the re-read fails too.
  Acceptance gains a row saying a read error that clears on the re-read lets the loop step on.
- **F5** (fixed as proposed): the unknown-key criterion is deleted.
  New criteria check D2's PATTERN line and background, and D7's cobra bullet, overview pointer and rubric check, with `lint-pattern` kept as a further condition.
- **F6** (fixed as proposed): D5's second Premise sentence is deleted.
  The Auto-mode assumption that referred to "D5's root-cause attribution" now points at support-log.md Q5, since D5 no longer holds the attribution.

## Disputed

None.

## Deferred

None.
