# Card 3 — stencilstore-forward-refresh

**Edit:**
- `internal/stencilstore#Mode`
- `internal/stencilstore#ModeProduction`
- `internal/stencilstore#ModeDev`
- `internal/stencilstore#ModeFor` -> `func ModeFor(dev, production, cleanStamp bool) Mode`
- `internal/stencilstore#Source`
- `internal/stencilstore#Reconcile`
- `internal/stencilstore#reconcileOne` -> `func reconcileOne(path, name string, state State, onDisk, shipped []byte, mode Mode, source Source, force bool) (bool, error)`
- `internal/stencilstore#writeStamped` -> `func writeStamped(path string, shipped []byte, hash string, writer Writer) error`
- `internal/stencilstore#ForceRefresh` -> `func ForceRefresh(baseDir string, registry Registry, mode Mode, source Source) ([]string, error)`
- `internal/stencilstore/stencilstore_test.go`
- `internal/stencilstore/reconcile_test.go`
- `internal/stencilstore/modefor_test.go`
- `internal/stencilstore/doc.go`
- `cmd/lyx#seedStencilsAt`
- `internal/cliwire#Module.ResolveStandalone`
- `internal/stencilcli#Command`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/stencilstore#ModeUnstamped` -> `const ModeUnstamped Mode = 0`
- `plan:internal/stencilstore#Writer` -> `type Writer struct`
- `plan:internal/stencilstore#Ordering` -> `type Ordering int`
- `plan:internal/stencilstore#OrderingUnknown` -> `const OrderingUnknown Ordering = 0`
- `plan:internal/stencilstore#RecordedOlder` -> `const RecordedOlder Ordering = 1`
- `plan:internal/stencilstore#RecordedNotOlder` -> `const RecordedNotOlder Ordering = 2`
- `plan:internal/stencilstore#ApplyWriter` -> `func ApplyWriter(content []byte, writer Writer) []byte`
- `plan:internal/stencilstore#ParseWriter` -> `func ParseWriter(content []byte) Writer`
- `plan:internal/stencilstore#WritesDue` -> `func WritesDue(baseDir string, registry Registry, mode Mode, source Source) (bool, error)`

**Uses:**
- `plan:internal/buildinfo#IsProduction`
- `plan:internal/buildvcs#Identity.Clean`

**Intent:** D1 and D2 inside the store, which still runs no git (`PATTERN-stencil-ownership`).
`Mode`'s zero value becomes `ModeUnstamped`, with `ModeDev` and `ModeProduction` after it, so an unset mode never refreshes; `ModeFor` is the single three-way mapping: `ModeProduction` only when `production` and `cleanStamp` both hold, else `ModeDev` when `dev`, else `ModeUnstamped`.
Its callers pass `buildinfo.IsDev()`, `buildinfo.IsProduction()` and `buildvcs.Running().Clean()`: `seedStencilsAt` in the pre-run and `Module.ResolveStandalone`'s two calls in `cliwire`.
`Writer` is a recorded or running binary, `Revision` plus `Time`; `ApplyWriter` sets, rewrites or, for a zero writer, removes `build=<revision>` and `time=<RFC 3339>` keys after the hex in the existing `lyx-stencil:` banner line, never touching the body, and `ParseWriter` reads them back, zero when absent; `BodyHash`, `Classify`, `ApplyStamp` and `ParseStamp` are unchanged.
`Source` gains two fields: `Writer`, the running binary, and `Older func(recorded Writer) Ordering`, the caller's ordering read; a nil `Older` answers unknown.
`reconcileOne`'s untouched-differing row now reads the recorded writer: `ModeProduction` refreshes when the file carries no recorded revision or `Older` answers `RecordedOlder`, and otherwise leaves the file with one Info line naming the stencil, both revisions and `lyx stencil sync` as the override; `ModeDev` warns as today; `ModeUnstamped` seeds absent files and warns once per stencil naming `lyx stencil sync` and `update-plugins.sh`; `force` performs the refresh row whatever the mode or ordering.
Every write stamps the body hash through `writeStamped`, which records `source.Writer` only when `mode == ModeProduction` and a zero writer otherwise, so a dev, unstamped or forced non-production write leaves no recorded revision and the next production binary refreshes it once; the restamp row applies the same writer rule.
`Reconcile` and `ForceRefresh` share one decision step, extracted from `reconcileOne` so `WritesDue` runs the same classification over every registry name and `.gitattributes` without writing and reports whether any write is due; `ForceRefresh` takes the mode its caller computed and sets `force`, replacing today's hard-coded `ModeProduction`.
`Command` in `stencilcli` passes the mode from `ModeFor` to both `ForceRefresh` calls; nothing else in that verb changes on this card.
Tests, all against the fake registry and a bare temp dir: rows in `TestReconcile_UntouchedBoardCopy` for no recorded revision, `RecordedOlder`, `RecordedNotOlder` with the Info line, unknown ordering and nil `Older`; rows in `TestReconcile_ModeDevRefusalNamesItsRemedy` for the unstamped warning's two remedies and for the zero `Mode`; `TestForceRefresh_PerformsRefreshRowFromDevSkippedFile` asserts the writer is recorded under `ModeProduction` and absent under `ModeDev` and `ModeUnstamped`; `TestModeFor` becomes the full truth table; a new `TestApplyWriter_RoundTripsThroughParseWriter` table in `stencilstore_test.go` covers set, rewrite, remove and a banner with no keys; a new `TestWritesDue` covers no write due, an absent file, an untouched copy under each mode and an edited copy; the budget row for `internal/stencilstore` rises by two.
`doc.go` states the forward-only rule, the three modes and the writer keys; `docs/overview.md`'s stencil row waits for card 8, once the pre-run behavior is complete.

**ImpactSummary:** Re-signs four store functions whose callers are `cmd/lyx`, `cliwire`, `stencilcli` and the store's own tests, all listed; every other consumer of `Mode` constants and `Reconcile` keeps compiling, and writer keys are written only from card 4 on, once `StencilSource` fills `Source.Writer`.
