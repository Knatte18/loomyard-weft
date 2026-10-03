# Card 41 — envelope-assertions

**Edit:**
- `fabriccli/addleftover_integration_test.go`
- `fabriccli/envelopecontract_integration_test.go`
- `fabriccli/merge_cli_integration_test.go`
- `fabriccli/pushbypass_integration_test.go`
- `fabriccli/remoteenvelope_integration_test.go`
- `reedcli/cli_test.go`
- `reedcli/cli_integration_test.go`
- `reedcli/watchdog_integration_test.go`
- `//cmd/lyx/main_test.go`
- `//cmd/lyx/main_integration_test.go`
- `//cmd/lyx/exitcode_test.go`
- `//cmd/lyx/unknown_subcommand_test.go`

**Uses:**
- `testkit/envelope/envelope.go`

**Intent:**
Finish #324 §6's envelope item where #324 measured it: the fabric, reed and `cmd/lyx` tests that unmarshal an envelope by hand and assert `env["ok"].(bool)` / `env["error"].(string)` decode through `envelope.Decode` and assert with `RequireOK`/`RequireErr` or the typed `OK`/`Error`/`Partial` fields.

An assertion checking exact equality keeps exact equality (compare `Envelope.Error` with `==`), never weakened to `RequireErr`'s substring.
Module-specific keys are read from `Raw`.
A test whose subject is the raw JSON bytes (key order, absence of a key, an exit-code mapping) keeps its own check on those bytes.
The envelope-contract tests are D9-protected regression tests: restructure only, every assertion survives.

Other CLI packages with inline `["ok"]` assertions may follow later; this card covers the measured set.
Apply D10 to every touched test.

**ImpactSummary:** fabriccli, reedcli and cmd/lyx tests switch hand-rolled envelope decoding to the kit; every assertion keeps its strictness.
