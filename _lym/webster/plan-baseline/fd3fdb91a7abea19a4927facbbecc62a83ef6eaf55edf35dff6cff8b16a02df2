# Card 19 — treadle-delete

**Create:**
- `plan:internal/agentconfig#RetiredRolesReinterpretation` -> `const RetiredRolesReinterpretation = "agents-treadle-roles-retired"`
- `plan:internal/agentconfig#DropRetiredRoles` -> `func DropRetiredRoles(existing []byte, worktreeRoot, anchorDir string) ([]byte, []string, []string, error)`
- `plan:internal/agentconfig_test#TestDropRetiredRoles` -> `func TestDropRetiredRoles(t *testing.T)`

**Delete:**
- `internal/treadleengine#`
- `contracts/stencils/treadle/treadle-template-judge-circling.md`
- `contracts/stencils/treadle/treadle-template-judge-milestone.md`
- `contracts/stencils/treadle/treadle-template-targeting.md`
- `contracts/stencils#TreadleTemplateJudgeCircling`
- `contracts/stencils#TreadleTemplateJudgeMilestone`
- `contracts/stencils#TreadleTemplateTargeting`
- `internal/agentconfig#RoleTreadleJudge`
- `internal/agentconfig#RoleTreadleTargeting`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `internal/agentconfig#roles`
- `internal/agentconfig#skillLists`
- `internal/agentconfig#checkEfforts`
- `internal/mergeresolve#mergeresolveAllowedImports`
- `internal/agentconfig_test#TestLoad_RefusesUnusableContent`
- `internal/agentconfig/template.yaml`
- `internal/configreg#Modules`
- `internal/configreg#TestModules_RegisterTheirReinterpretation`
- `internal/verifyrun#waitDelay`
- `internal/webstercli/sync.go`
- `internal/shedengine#TestProducerSeamInvariant_AllowlistOnly`
- `.gitattributes`
- `pattern/PATTERN-cwd-resolution.md`

**Extends:**
- `internal/agentconfig/doc.go`

**Uses:**
- `internal/configreg#Reinterpretation`
- `internal/shuttleengine#DropEditLoopEdits`
- `internal/configreg/doc.go`
- `internal/webstercli/doc.go`

**Intent:**
Retire treadle (D8, and the operator's 2026-11-18 decision recorded under Added after Discussion).

Delete the package `internal/treadleengine` whole, with its tests, and the three stencils under `contracts/stencils/treadle/`, their embed vars, their `entries` rows and their `roleOpeningStencils` keys, and their `.gitattributes` lines.
`agentconfig` loses `RoleTreadleJudge`, `RoleTreadleTargeting` and their `roles` entries, and its `template.yaml` loses the two skill lists.

A hub `agents.yaml` still naming a retired role would stop loading, since `agentconfig.Load` refuses a `skills` key naming a role lym does not start.
`DropRetiredRoles` is the `agents` module's reinterpretation, modelled on `shuttleengine.DropEditLoopEdits`: it removes the `treadle-judge` and `treadle-targeting` keys from `skills` and every `efforts` key whose role part names either, carrying every other key and comment through, reports one rewrote line per removed key, and returns a document that does not parse marked `configengine.ErrInvalid`.
`configreg.Modules`' `agents` row registers it under `RetiredRolesReinterpretation`, so `hubreconcile`'s walk applies it once per hub.

The reinterpretation rewrites the hub's base file; a worktree or local `agents.yaml` layer naming a retired role still fails the load.
So `skillLists`' refusal of a `skills` key naming no role lym starts, and `checkEfforts`' refusal of an `efforts` key whose role part names no role lym starts, each name "way forward: remove the key from that layer's agents.yaml with lym config agents" in place of the generic way forward that sends the fix to the prime.
An `efforts` key is CategoryNone, so a task's `run_settings` can put one into the local layer.

`TestDropRetiredRoles` is tier0: a file naming `treadle-judge` in `skills`, `treadle-targeting/opus` and the bare `treadle-judge` in `efforts` is rewritten without them, and the result then loads through `agentconfig.Load`; a file naming neither comes back unchanged; an unparsable one is refused as invalid.
`TestLoad_RefusesUnusableContent`'s unknown-role rows assert the way forward, and it gains a row for an `efforts` key naming `treadle-judge/opus` asserting the same way forward.
`TestModules_RegisterTheirReinterpretation` gains the `agents` row.

The comments naming treadle are reworded to stand alone: `waitDelay`'s, `sync.go`'s file comment listing `.lym` runtime artifacts, the header of `TestProducerSeamInvariant_AllowlistOnly`'s file, which compares its allowlist with treadle's,
and the doc comment of `mergeresolveAllowedImports`, which cites the call treadleengine's seam allowlist makes.
`PATTERN-cwd-resolution.md` drops `treadleengine` from its bound packages.

**ImpactSummary:** Deletes a package with no importer left after card 18, three stencils and two roles; one reinterpretation keeps every hub agents.yaml loading.
