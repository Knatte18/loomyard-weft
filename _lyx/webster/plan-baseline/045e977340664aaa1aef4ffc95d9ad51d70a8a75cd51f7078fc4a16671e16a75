# Card 29 — wide-call-helpers

**Edit:**
- `internal/websterengine/template_test.go`
- `internal/shedrun/paths_test.go`
- `internal/loomcli/wiring_commitstatus_test.go`
- `internal/loomcli/wiring_commitstatus_integration_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/battencli/commitstatus_test.go`

**Intent:** Route each test file's calls to a wide production function through one local helper, so a signature change edits one line per file (D16).
No production signature changes.
In `template_test.go`, every `websterengine.RenderIntegrationPrompt` call goes through one helper taking only what the calls vary;
in each other listed file that still calls `shedrun.StatusRel` more than once after card 28, the calls go through one local helper.
A file left with a single call keeps it direct.
`wiring_commitstatus_integration_test.go` is protected (D21): only its call spelling changes, never an assertion.
`smoke_test.go` is a `smoke`-tagged file, so this card runs `loomcli`'s smoke tier (D21);
a flaky result is re-run, not waved through.

**ImpactSummary:** Test call sites only; no production signature or assertion changes.

**Verify:** go test -tags smoke ./internal/loomcli/
