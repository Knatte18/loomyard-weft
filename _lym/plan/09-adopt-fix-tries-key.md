# Card 9 — beam.yaml's adopt_fix_tries

**Edit:**
- `internal/dobbyengine#Config`
- `internal/dobbyengine#ConfigSpec`
- `internal/dobbyengine#LoadConfig`
- `internal/dobbyengine#TestLoadConfig`
- `internal/dobbyengine/template.yaml`
- `internal/dobbyengine/doc.go`

**Uses:**
- `internal/dobbyengine#writeBeamLayer`

**Intent:**
D6: the number of fix tasks a failed adopt check files for its subject between two hand adopts is the new `beam.yaml` key `adopt_fix_tries`, template default 3, read through `configengine` with the `beam` module's template (PATTERN-config-layers).

`template.yaml` gains `adopt_fix_tries: 3` with its comment: the most adopt fix tasks a failed adopt check files for the worktree's subject between two hand adopts; 0 files none.
`Config` gains `AdoptFixTries int` (`yaml:"adopt_fix_tries"`); `ConfigSpec` gives it `configengine.CategoryNone`; `LoadConfig` refuses a negative value naming the key through its existing negative-value loop.
`configreg` registers the module through `ConfigSpec`, so no registry edit is needed.

`TestLoadConfig` gains rows: the template reads 3; a layer setting 1 reads 1; a negative value is refused naming `adopt_fix_tries`.

`doc.go`'s Config paragraph names `adopt_fix_tries` as the adopt check's try limit, which dobby itself never reads.

**ImpactSummary:** one new key on the beam module's template, Config and spec; no reader of the existing keys changes.

**Commit:** 9: beam.yaml: adopt_fix_tries, the adopt fix loop's try limit
