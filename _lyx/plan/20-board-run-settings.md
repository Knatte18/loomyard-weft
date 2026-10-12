# Card 20 — board run settings

**Create:**
- `plan:internal/boardengine#RunStatusProducer` -> `func RunStatusProducer(status *string) (string, bool)`
- `plan:internal/boardengine_test#TestRunStatusProducer` -> `func TestRunStatusProducer(t *testing.T)`
- `plan:internal/boardcli#prepareRunSettings` -> `func prepareRunSettings(b *boardengine.Board, fields map[string]any, strandName string) (string, error)`
- `plan:internal/boardcli_test#stepRunSettings` -> `func stepRunSettings(t *testing.T, f *cliFixture)`

**Edit:**
- `internal/boardengine#Task`
- `internal/boardengine#UpsertPayloadKeys`
- `internal/boardengine#normalizeTask`
- `internal/boardengine#scopeChanged`
- `internal/boardengine#cloneTasks`
- `internal/boardengine_test#TestUpsertFieldAllowlist`
- `internal/boardengine_test#TestApplyPatch`
- `internal/boardengine_test#TestRunLock`
- `internal/boardengine/doc.go`
- `internal/boardcli#storeVerbs`
- `internal/boardcli#Command`
- `internal/boardcli#outputSuccessWithTask` -> `func outputSuccessWithTask(out io.Writer, task boardengine.Task, notice string) int`
- `internal/boardcli_test#TestCLI`
- `internal/boardcli/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/configreg#PrepareRunSettings`
- `plan:internal/agentname#TaskSlug`

**Intent:**
Add D8's `run_settings` to the board task.

In boardengine: `Task` gains `RunSettings map[string]any` with JSON key `run_settings`, omitted when empty: an object of module name to a mapping of that module's keys, nested as in the YAML file.
`UpsertPayloadKeys` lists `run_settings` as optional, so upsert, upsert-batch and merge's nested upsert accept it; `normalizeTask` stores an empty object as no field, so `"run_settings": {}` clears it.
A `run_settings` change is not a scope change: `scopeChanged` never compares it and its doc says so, so a run-held entry takes it, and `cloneTasks` copies the map so the run-lock snapshot cannot alias it.
`RunStatusProducer` returns the producer half of a status `IsRunStatus` accepts, and false otherwise.
`doc.go` gains a "Run settings" section: the shape, the clearing rule, that a change never counts as scope, and that `lyx board intake` never sets the field.

In boardcli: `prepareRunSettings` runs on every payload that carries `run_settings`, before the store write, in upsert, in each task of upsert-batch and in merge's upsert, through `storeVerbs`:

- the value must be an object, and `configreg.PrepareRunSettings` migrates and validates it; a refusal is the verb's error, naming the module and the key; the migrated documents are decoded back and replace the payload's value, so the board stores the migrated result;
- when the value differs from the task's stored one (read through `Board.GetTask` before the write), a session whose strand name (`agentname.StrandNameEnv`) yields a slug from `agentname.TaskSlug`, the same guard the hub-wide config write uses, is refused, naming the operator or the hub orch as the way forward; the bound is the guard's own, a mistaken write and not a determined one, read outside the write lock;
- when the value differs and the stored status is a run status whose producer is neither `Worktree-Create` nor `Seed-Child` (`battenrecipe.NameWorktreeCreate`, `battenrecipe.NameSeedChild`), it returns the notice that the change applies to the next run's Seed-Child and that the running pair takes it through `lyx config <module> --layer local --set` in the pair, which `outputSuccessWithTask` adds as a `notice` field of the envelope, an empty notice omitting the field, as `Command`'s promote verb passes (upsert-batch and merge carry the notices of their tasks the same way).

Value semantics, such as a positive integer, stay with the module's load.
The upsert command's `Use` gains `run_settings?` and its `Long` documents the field, its validation, the guard and the notice; boardcli's `doc.go` says the same.

`TestRunStatusProducer` pins a run status, `done`, a hand-set word and nil; `TestUpsertFieldAllowlist` accepts `run_settings`; `TestApplyPatch` pins `{}` clearing it; `TestRunLock` pins a `run_settings` change on a run-held entry passing while a title change is still refused.
`stepRunSettings`, run from `TestCLI`, drives the verbs: a valid loom setting is stored migrated; a non-object value (`"run_settings": 3`) is refused naming `run_settings`; an unknown module, a key absent from the template outside open maps and aliases, a shape mismatch and a D4-closed key are each refused naming module and key; a slug-named session's change is refused while an identical re-write passes; a change on a task whose status is `running · Discussion-Write` returns the notice, and one at `running · Seed-Child` returns none.
The budget row for `internal/boardengine` rises by one.

**ImpactSummary:** one new task field and its validation on every upsert path; the run lock and README are unchanged, and the upsert envelope gains an optional `notice`.
