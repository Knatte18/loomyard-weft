# Lens review

Every lens returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates merged: coverage F1, guard-reach F1 and way-forward F5 are one finding (the failed-builder branch), recorded under guard-reach.

## coverage

- coverage F1 (MEDIUM), the builder-failure branch of `Seats` and `LookupSeat` is unreached: merged into guard-reach F1, fixed as proposed there.
- coverage F2 (MEDIUM), the verb's unmet-requirement exit has no CLI row: fixed as proposed. Card 8's `TestManifestCmd` gains a row for a seat with an unmet Codex requirement that names `codex` and the requirement and exits non-zero.
- coverage F3 (LOW), `copyManifest`'s unreadable-record branch has no row: fixed as proposed. Card 2's `TestRunner_Start_KeepsTheManifestCopy` gains a row whose hook writes a `manifest.json` that is not JSON: the run finishes done with no copy.
- coverage F4 (LOW), no row asserts the configured sandbox mode under `ConfiguredSandbox`: fixed as proposed. Card 6's fixture sets `sandbox_mode: auto`, and `TestSeats_CodexManifestHoldsClaudes` asserts `workspace-write` in each compared Codex manifest's sandbox `settings` item.
- coverage F5 (LOW), the method-glyph and function-literal rules have no temp-dir row: fixed as proposed. Card 7's `TestLaunchSpecLiterals_AreListed` gains both rows.
- Note outside the lenses (guard-reach and way-forward): seats and producers with any requirement, background wake included, are not compared, because Codex implements no `RequirementSupporter` (`internal/shuttleengine/requires.go:94-101`, `internal/shuttleengine/claudeengine/capabilities.go:16` the only implementer). `Start` refuses such a spec on Codex, so no Codex launch exists to compare. No card changes; the overview's decision now names background wake beside forks.

## row-validity

- row-validity F1 (MEDIUM), `TestReport` and `TestManifestCmd` cannot inject a fixture catalog, and `shuttlecli` cannot reach `manifestEngine`: fixed differently. The proposal was a `SeatEnv` field naming the tables plus a manifest-writing fake engine for `shuttlecli`. An exported field of the unexported `seatTable` type cannot be set from outside the package. A fake engine in `shuttlecli` would make that test re-run what `TestReport` already covers. Instead, card 6 adds `seatsOf(env, tables)`, which `Seats` calls on `seatTables`, and card 8 adds `report(env, tables, …)`, which `Report` calls, and `TestReport` runs it on fixture tables. Card 8 also replaces the `manifestEnv` field with `manifestReport func(name string) (ManifestReport, error)`, which `TestManifestCmd` stubs with hand-built reports.
- row-validity F2 (LOW), card 4's `ErrHookUnrun` rows: fixed as proposed. The no-warning row moves to `TestPrepare_UserSetup`, which card 4 now lists under Edit. The other-reason row is dropped because `OmittedHookAndWarningReachTheWarnings` already pins it. "warn log" now reads "`warnings` buffer".
- row-validity F3 (LOW), card 3's fixture helpers are unlisted and the plugin scope is unstated: fixed as proposed. `writePluginInstalls`, `writeUserSetupFile` and `writeSkillFile` join Uses. The row says the plugin is a user-scope install whose skill the spec loads, so the scope turns it on.
- row-validity F4 (LOW), Claude's unrun plugin hooks could become omitted items unnoticed: fixed as proposed. Card 3's autonomous row asserts the plugin context is `native` and that the record's `Omitted` verdict is empty.

## forced-edits

- forced-edits F1 (LOW), card 5 does not list `constructor`: fixed as proposed, with the result copied from the declaration `lym quarry resolve` gives (`type constructor func(cfg shuttleengine.Config, dirs configengine.Dirs) shuttleengine.Engine`, with no error result). Card 5 lists `internal/shuttleengines#constructor` -> `type constructor func(cfg shuttleengine.Config, dirs configengine.Dirs, seams LaunchSeams) shuttleengine.Engine`.

## guard-reach

- guard-reach F1 (MEDIUM), `LookupSeat`'s failed-builder refusal is unreachable because a failed builder yields no seat names. This merges coverage F1 and way-forward F5, and is fixed as proposed. Card 8 drops that branch. The refusal for a name no built seat holds names the catalog's seats and every builder error. `TestReport` gains a row with a failing fixture `Build`: the other entry's seat still reports, and the failing entry's seat name is refused naming that builder.
- guard-reach F2 (LOW), card 4's text match depends on `UnrunHooks` returning `ErrHookUnrun` unwrapped: fixed differently. The proposal was an end-to-end no-warning assertion in card 5's or card 6's test. Card 6 instead states that `UnrunHooks` never wraps the error, and why, and `TestCompare` gains an `UnrunHooks` row asserting `==` against `ErrHookUnrun`. Card 4's `TestPrepare_UserSetup` row pins the warning skip on that exact text. Together the two rows fail if either link breaks, and neither needs the real hook path to emit a warning.

## way-forward

- way-forward F1 (MEDIUM), the verb's unmet-requirement exit names no next step: fixed as proposed. Card 8's exit says the seat cannot run on that engine until its engine package meets each requirement through `shuttleengine.RequirementSupporter`, that no bench of the seat runs on it, and that the other engine's printed manifest still stands.
- way-forward F2 (LOW), the Table-literal way forward could file a new builder under an unrelated entry: fixed as proposed. Card 7's message tells a new builder to get its own `seatTables` entry with `Builder`, `Holders` and `Build`, and a helper to go under its builder's `Holders`.
- way-forward F3 (LOW), the failed-copy warning names no next step: fixed as proposed. Card 2's warning names the copy path and the error, and says to remove whatever blocks `<root>/manifests`.
- way-forward F4 (LOW), the parity test's builder failure names no next step: fixed as proposed. Card 6's `seatsOf` errors name the builder and say to fix the stand-in inputs its `seatTables` entry's `Build` writes, and the test prints that error.
- way-forward F5 (LOW), a failed builder's own way forward points at a run's input, which the verb's operator cannot follow: merged into guard-reach F1 and fixed by the same card 6 error text, which the verb's refusal relays.
