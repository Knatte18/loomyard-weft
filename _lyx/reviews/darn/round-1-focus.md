---
round: 1
focus:
  - "Scope against the board entry: read the board entry for `reed-launch-fixes` and settle whether the diff makes every change it asks for (issues #627 and #628) and nothing it does not license."
  - "`newAgentWindowSelected` and `render.Display.KeepViewer`: settle whether the zero value keeps the follow-the-run behavior loom's driver, arbiter and seat spawns rely on, and whether `Focus` wins over `KeepViewer` in every branch."
  - "`reed.json` persistence of `keepViewer`: settle whether the field round-trips, is omitted when false, and is carried wherever `Display` is copied or rebuilt, such as `lyx reed add` and the shuttle spawn path."
  - "`launchStrandLocked` and `shell.Shell.DiscardLineKey`: settle whether the discard key is sent as a key and not as literal text, whether the empty-key (pwsh) path types only the payload and `Enter`, and whether any other launch path needs the same key."
  - "Repository rules: settle whether `doc.go` for the touched packages, `PATTERN.md` and any CLI help text changed in the same diff as the observable `--focus=false` behavior change, per CLAUDE.md's docs rule."
  - "Change description claims in `_lyx/landing/summary.md`: check each claim, including the `@lyx_unseen` behavior and the renamed tests, against the diff."
  - "Test economy: settle whether any new case duplicates an existing test's coverage, naming the covering test, and whether any added behavior has no test."
---

Seed focus only; the base diff review still applies in full.
