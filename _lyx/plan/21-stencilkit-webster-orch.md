# Card 21 — stencilkit-webster-orch

**Edit:**
- `internal/websterengine/template_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/burlerengine/prompt_test.go`
- `internal/treadleengine/judge_test.go`
- `internal/orchengine/prompt_test.go`
- `internal/orchcli/start_test.go`

**Uses:**
- `internal/testkit/stencilkit/stencilkit.go`

**Intent:** Convert these packages' stencil seeders to `stencilkit` (D11): `websterengine`'s `newTestStencilsDir`, `seedHubWebsterStencils`, `seedHubPatternStencils`, `seedHubStencils`, `seedHubFrictionStencils` and `patternActiveMissingPatternStencilsLayout`;
`webstercli`'s `seedHubStencils`;
`burlerengine`'s `newTestStencilsDir`;
`treadleengine`'s `newTestStencilsDir`;
`orchengine`'s `seedStencils`;
`orchcli`'s `seedStartStencils`.
A deliberate-absence fixture becomes "seed all, then remove X".
A test that compared seeded bytes against an embedded default compares stamp-stripped bodies instead;
a verdict that changes because the seeded file now carries the stamp is reported as a finding, not absorbed by a fixture tweak.
Fixtures that write deliberately modified stencil content keep writing it, on top of a `stencilkit` seed.
The `smoke`-tagged seeders already reconcile through the registry and are left as they are, so no smoke file changes.
After this card none of these files hand-lists stencil byte slices to seed a directory.

**ImpactSummary:** Test fixtures only, in six packages; every converted test keeps its assertions.
