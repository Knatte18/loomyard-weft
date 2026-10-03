# Card 40 — envelope-decoders

**Edit:**
- `fabriccli/envelope_test.go`
- `fabriccli/cli_test.go`
- `fabriccli/merge_cli_integration_test.go`
- `fabriccli/mergein_webster_integration_test.go`
- `fabriccli/pushbypass_integration_test.go`
- `fabriccli/status_mergeinprogress_integration_test.go`
- `shedcli/parity_test.go`
- `selfreportcli/cli_test.go`
- `loomcli/validate_test.go`
- `loomcli/parity_test.go`
- `loomcli/review_test.go`
- `quarrycli/verbs_test.go`

**Uses:**
- `testkit/envelope/envelope.go`

**Intent:**
Replace the per-package JSON envelope decoders with card 39's `envelope` kit (#324 §6, D2).

Delete `decodeEnvelope` (`fabriccli/envelope_test.go`, `shedcli/parity_test.go`), `decodeResult` (`fabriccli/cli_test.go`), `parseEnvelope` (`selfreportcli/cli_test.go`), `decodeSingleEnvelope` (`loomcli/validate_test.go`) and `decodeReview` (`loomcli/review_test.go`); every caller, in the defining file or a sibling file listed here, uses `envelope.Decode` and reads `Raw` for module-specific keys.
`quarrycli`'s `decodeDocuments` converts if its documents are envelopes and stays otherwise; `loomcli`'s typed `decodeMidMergeEnvelope` stays, since it decodes a module-specific struct.
Every caller of a deleted decoder reads the returned `Envelope`'s typed fields or `Raw` in place of the old map, and an `env["ok"].(bool)` or `env["error"].(string)` check right after decoding becomes `RequireOK`/`RequireErr` or a typed-field comparison.
Envelopes those files unmarshal by hand, without a decoder helper, are card 41's.
An assertion checking exact equality keeps exact equality, never weakened to `RequireErr`'s substring.
Apply D10 to every touched test.

**ImpactSummary:** CLI test files in five packages drop their own decoders for the kit; assertions keep their meaning with typed access.
