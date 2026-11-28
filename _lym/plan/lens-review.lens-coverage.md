```yaml
lens: coverage
outcome: reported
```

Coverage lens findings for `_lym/plan`:

- MEDIUM — card 6 (`Seats`) and card 8 (`LookupSeat`): no planned row reaches the branch where a builder fails. `Seats` returns one error per failing builder, and `LookupSeat` refuses a seat whose builder failed by naming that builder's error. The rows don't get there:
  - `TestSeats_CodexManifestHoldsClaudes` fails on any builder error, so it never exercises a builder failing.
  - `TestReport`'s rows are: match, missing item, unrun context, unmet requirement, unknown seat, `Prepare` failing, and no record written. None has a failing builder.
  - Breaking either branch, for example letting one builder's error block every seat, would pass every planned test.
  - Evidence: card 6 Intent ("It returns beside them one error per builder that failed"), card 8 Intent ("A name whose builder failed is refused naming that builder's error"), and the card 8 `TestReport` row list.
- MEDIUM — card 8 (`shuttleCLI.manifestCmd`): the CLI exit for an engine that can't start the seat (`output.Err` naming the engine and its unmet requirements, non-zero exit) has no row.
  - `TestManifestCmd` covers the matching seat, the missing Codex item and the unknown seat.
  - `Unmet` is covered only one level down, in `TestReport`.
  - Evidence: card 8 Intent, the third bullet of the verb's exits, against the `TestManifestCmd` rows.
- LOW — card 2 (`copyManifest`): the branch where a record can't be read (logged at Warn, launch continues) has no row. The only failure row is the one where the copy can't be written (a regular file at `&lt;root&gt;/manifests`). Evidence: card 2 Intent ("A record that cannot be read or a copy that fails to write is logged at Warn").
- LOW — card 6 (`ConfiguredSandbox`): D10 says the probe "reports the configured mode", and the card says `auto` and `workspace-write` settle to `workspace-write`. No row asserts the mode the Codex `settings` item lists under `ConfiguredSandbox`. Card 5's test uses a probe that records its calls, not `ConfiguredSandbox`. Evidence: D10 in the decision record and card 6's "Shared seams" bullet.
- LOW — card 7 (`unlistedLiterals`): the rule that a method is spelled `&lt;dir&gt;#&lt;Receiver&gt;.&lt;Method&gt;`, and that a literal inside a function literal takes the glyph of its enclosing named function, has no temp-dir row. Only the `RealModule` row exercises it. Evidence: card 7 Intent and its `TestLaunchSpecLiterals_AreListed` row list.

Tool calls:
- grep of `internal/shedadapters/bouncer.go` for `Prompt:` and the bouncer render methods, to check card 7's claim that `judgeCall` and `runSeedSpawn` already render through engine-taking methods. My context held only the glyphs, not the bodies.
