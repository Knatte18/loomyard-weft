# Card 1 — engine-set

**Create:**
- `plan:internal/shuttleengine#EngineSet` -> `type EngineSet struct`
- `plan:internal/shuttleengine#NewEngineSet` -> `func NewEngineSet(defaultName string, engines map[string]Engine) EngineSet`
- `plan:internal/shuttleengine#EngineSet.Lookup` -> `func (s EngineSet) Lookup(name string) (Engine, error)`
- `plan:internal/shuttleengine#EngineSet.Names` -> `func (s EngineSet) Names() []string`
- `plan:internal/shuttleengine#EngineSet.DefaultName` -> `func (s EngineSet) DefaultName() string`
- `plan:internal/shuttleengine#ErrEngineNotRegistered` -> `var ErrEngineNotRegistered = errors.New("shuttle: the engine is not registered")`
- `plan:internal/testkit/shuttlefake#EngineName` -> `const EngineName = "fake"`
- `plan:internal/testkit/shuttlefake#EngineSet` -> `func EngineSet(engine shuttleengine.Engine) shuttleengine.EngineSet`
- `internal/shuttleengine/engineset_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Add the engine set that shuttle's runner and readers will take in place of one `Engine` (D2), in a new file `internal/shuttleengine/engineset.go`.

`EngineSet` holds a name-to-`Engine` map and the default engine's name; the zero value is an empty set.
`NewEngineSet` copies the map it is handed and stays total, so a caller with a bad default learns it at `Lookup`.
`Lookup("")` resolves the default name, so a spec with no engine and a run record with no `engine` key (D5) both read as the default.
`Lookup` of a name the set does not hold, including a default name that is not registered, returns an error wrapping `ErrEngineNotRegistered` that names the unknown name and the registered names, sorted, comma-separated.
`Names` returns the registered names sorted; `DefaultName` returns the default name as told.

In `internal/testkit/shuttlefake`, `EngineSet` wraps one engine as a set registered under `EngineName`, which is also its default, so the consumer tests that later cards move onto the set write one call.
No test of the helper itself: the consumer tests later cards edit reach it.

`internal/shuttleengine/engineset_test.go` adds `TestEngineSet_Lookup`, table-driven: an empty name resolves the default; a registered name resolves its own engine; an unknown name and a set whose default is unregistered each fail with `errors.Is(err, ErrEngineNotRegistered)` and an exact message naming the sorted registered names; `Names` is sorted; mutating the map after `NewEngineSet` does not change the set.
Raise `internal/shuttleengine` in `test-budget.yaml` from 172 to 173.

**ImpactSummary:** New types and functions only; the one edit raises the shuttleengine row of the test budget by one.
