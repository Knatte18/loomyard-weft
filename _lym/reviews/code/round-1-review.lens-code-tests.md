```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it. One gap found.

**Per behaviour:**

| Behaviour | Pinning test |
|---|---|
| `output.ErrorEnvelopeLine` accepts a whole-line envelope: bare, indented under a heading, with extra fields | `TestErrorEnvelopeLine` rows BareEnvelope, EnvelopeUnderAHeading, EnvelopeWithFields (literal expectations) |
| `ErrorEnvelopeLine` passes prose, a success envelope, `ok:false` with no error, an error with no ok, a non-string error, an envelope inside a prose line, and empty text | `TestErrorEnvelopeLine` rows PlainProse, SuccessEnvelope, FalseOkWithoutError, ErrorWithoutOk, NonStringError, EnvelopeInsideProseLine, Empty |
| `applyBodyFile` refuses a file holding an envelope, names it, and leaves `fields` without `body` | `TestApplyBodyFile` row "a file holding a CLI error envelope is refused naming the envelope" (`internal/boardcli/bodyfile_test.go:83-90`) |
| The same refusal on the stdin route, and through upsert and merge `--body-file` | Same `applyBodyFile` code after the read; the existing `--body-file` wiring is pinned by `internal/boardcli/verbs_test.go:634-679`. No separate row needed. |
| Record-Write returns Stuck on an envelope body: the reason names the envelope, `batten cancel` and `board upsert --body-file`; no record is written and nothing is committed | `TestRecordWrite_Call/ErrorEnvelopeBodyIsStuckNamingTheEnvelope` (`internal/loomshed/recordwrite_test.go:272-289`) |
| The `cancelErr` call in the new Stuck branch | none, but the missing-entry branch above it has no such test either, and `CancelledContextWins` (`recordwrite_test.go:310`) only reaches `entryErr`. That is symmetric with the base, so no finding. |
| `resolveLookup` reads a bare slug as `{"slug":…}` (get, and through the shared path every lookup verb) | `TestCLI` `stepLookupContract` row `get_by_bare_slug` (`internal/boardcli/cli_test.go:858-874`) |
| `isBareSlug` rejects a non-slug shape (leading digit, space, punctuation, JSON literal) so it still reaches the JSON decode and its `invalid json` error | none (finding 1) |
| loomshed import allowlist gains `internal/output` | `TestToldGeometryInvariant_AllowlistOnly` (`internal/loomshed/seam_enforcement_test.go`) |
| `payloadLegend`, the `get` help example, and the `doc.go` and comment prose | static prose, exempt |

The changed `TestApplyBodyFile` doc comment still states what the table checks. Every expected value in the new tests is a literal, so none is tautological.

**Findings:**

1. **MEDIUM · class `design` · `internal/boardcli/cli.go:1221` (`isBareSlug`).** No test pins the rejecting side of `isBareSlug`.
   - The only new row, `get_by_bare_slug` (`internal/boardcli/cli_test.go:858-874`), checks acceptance.
   - The only bare non-JSON lookup refusal in the suite is `"invalid json": `{`` in `stepPromote_Refusals` (`internal/boardcli/verbs_test.go:120`). It asserts only `ok=false`, which an unknown-slug refusal also gives.
   - So a mutant that drops the `i == 0` leading-letter check, accepts every rune, or reorders the handle and bare-slug branches would pass every test. Under such a mutant, `{`, `1abc` or `null` would read as a slug.
   - Fix: add rows to the lookup contract table for a bare payload that is not slug-shaped. Use a leading digit such as `1abc`, a space such as `my task`, and `{`. Assert the exact error `invalid json: …` rather than only `ok=false`.

Tool calls:
1. Grepped `internal/boardcli/cli_test.go` for lookup payload rows and `internal/loomshed/recordwrite_test.go` for cancel cases. My context did not have the existing test tables.
2. Grepped the boardcli tests for `invalid json`, `resolveLookup` and `--body-file` usage, to find any test that pins the rejecting side and the body-file wiring.
3. Read `internal/boardcli/verbs_test.go:105-140`, to see what the `invalid json` row asserts.
