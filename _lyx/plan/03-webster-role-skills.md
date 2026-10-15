# Card 3 — webster-role-skills

**Edit:**
- `internal/websterengine#RunDeps`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine#Run`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#TestRun_MasterSpawn`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`

**Uses:**
- `plan:internal/languageconfig#AppendSkills`

**Intent:**
The Webster master and recovery strand load the told language list after `roleSkills` (D4).

`RunDeps` and `RecoverDeps` each gain the field `LanguageSkills []string`, documented as the hub's language skills the master (respectively the recovery strand) loads after `roleSkills`, empty for none.
`Run` sets the master spec's `Skills` to `languageconfig.AppendSkills(roleSkills, deps.LanguageSkills)`, and `recoverSpawn` sets the recovery spec's `Skills` the same way from its own deps.
`roleSkills` stays a package variable and forks still get no skills: they inherit the master's.

`internal/websterengine/doc.go`'s skills sentence adds that both spawns load the hub's language skills after `roleSkills`, from the `RunDeps.LanguageSkills` and `RecoverDeps.LanguageSkills` their wiring loads.

`TestRun_MasterSpawn`'s "the strand role, the model and the skills" row keeps its `roleSkills` assertion for a fixture that tells no list, and a new row prepares `fx.Deps.LanguageSkills` as `scribe:golang-testing`, `scribe:testing` and asserts the master spec's `Skills` equal `scribe:prose`, `scribe:code-quality`, `scribe:testing`, `scribe:golang-testing`.
`TestRecoverSpawnOrAttach` gains two rows over the spawning path the existing spawn rows use: one with no language list asserting `fx.Engine.LastSpec.Skills` equals `roleSkills`' three skills, and one with `fx.Deps.LanguageSkills` `scribe:golang-testing` asserting it equals those three followed by `scribe:golang-testing`.

**ImpactSummary:** two optional deps fields and two spec sites in `websterengine`; every deps literal (`internal/loomcli`, `internal/webstercli`, `internal/testkit/shedfake`, the package's own tests) compiles unchanged with an empty list, and cards 5 and 6 fill it.
