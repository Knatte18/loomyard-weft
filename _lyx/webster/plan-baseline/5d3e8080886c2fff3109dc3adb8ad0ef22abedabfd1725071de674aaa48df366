# Card 39 — envelope-kit

**Create:**
- `testkit/envelope/envelope.go`
- `testkit/envelope/envelope_test.go`

**Uses:**
- `internal/output#Ok`
- `internal/output#Err`
- `testkit/doc.go`

**Intent:**
Provide one shared JSON envelope decoder (#324 §6, D2), the decoder #324 placed at `internal/output/outputtest`, under `internal/testkit` so the Testkit Invariant covers it; card 40 moves the per-package decoders onto it.

`envelope.go` imports only stdlib and exports:

- `Decode(t testing.TB, out string) Envelope`, `t.Fatalf`ing on malformed JSON or more than one top-level document, where `Envelope` has typed `OK bool`, `Error string`, `Partial *bool` and `Raw map[string]any` for every other key;
- `RequireOK(t, out) Envelope` and `RequireErr(t, out, substr string) Envelope`, which fatal on the wrong `ok` value or a missing substring.

`RequireOK`/`RequireErr` are the second helper pair the Testkit Invariant's review discipline admits beside `shedfake`'s `Call`/`RequireOutcome` (card 1 records both); they check only the envelope's `ok`/`error` shape, never module behaviour.
Its package doc (D11) says the kit decodes the `internal/output` JSON envelope a CLI prints, imports only stdlib, and asserts nothing beyond the `RequireOK`/`RequireErr` shape check.

`envelope_test.go` covers decoding `output.Ok`/`output.Err`/`output.ErrFields` output, `Partial` present and absent, and the malformed and multi-document fatal paths through a recording `testing.TB` stub.
