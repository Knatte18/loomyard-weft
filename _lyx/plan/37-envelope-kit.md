# Card 37 — envelope-kit

**Create:**
- `testkit/envelope/envelope.go`
- `testkit/envelope/envelope_test.go`

**Edit:**
- `fabriccli/envelope_test.go`
- `fabriccli/cli_test.go`
- `shedcli/parity_test.go`
- `selfreportcli/cli_test.go`
- `loomcli/validate_test.go`
- `loomcli/review_test.go`
- `quarrycli/verbs_test.go`

**Uses:**
- `internal/output#Ok`
- `internal/output#Err`
- `testkit/doc.go`

**Intent:**
Replace the per-package JSON envelope decoders with one kit (#324 §6, D2), the decoder #324 placed at `internal/output/outputtest`, under `internal/testkit` so the Testkit Invariant covers it.

`envelope.go` imports only stdlib and exports:

- `Decode(t testing.TB, out string) Envelope`, `t.Fatalf`ing on malformed JSON or more than one top-level document, where `Envelope` has typed `OK bool`, `Error string`, `Partial *bool` and `Raw map[string]any` for every other key;
- `RequireOK(t, out) Envelope` and `RequireErr(t, out, substr string) Envelope`, which fatal on the wrong `ok` value or a missing substring.

`RequireOK`/`RequireErr` are the second helper pair the Testkit Invariant's review discipline admits beside `shedfake`'s `Call`/`RequireOutcome` (card 1 records both); they check only the envelope's `ok`/`error` shape, never module behaviour.

Delete `decodeEnvelope` (`fabriccli/envelope_test.go`, `shedcli/parity_test.go`), `decodeResult` (`fabriccli/cli_test.go`), `parseEnvelope` (`selfreportcli/cli_test.go`), `decodeSingleEnvelope` (`loomcli/validate_test.go`) and `decodeReview` (`loomcli/review_test.go`); callers use `envelope.Decode` and read `Raw` for module-specific keys.
`quarrycli`'s `decodeDocuments` converts if its documents are envelopes and stays otherwise; `loomcli`'s typed `decodeMidMergeEnvelope` stays, since it decodes a module-specific struct.
Call sites in these files that assert `env["ok"].(bool)` or `env["error"].(string)` right after decoding use `RequireOK`/`RequireErr` or the typed fields.

`envelope_test.go` (kit) covers decoding `output.Ok`/`output.Err`/`output.ErrFields` output, `Partial` present and absent, and the malformed and multi-document fatal paths through a recording `testing.TB` stub.
Apply D10 to every touched test.

**ImpactSummary:** CLI test files in five packages drop their own decoders for the kit; assertions keep their meaning with typed access.
