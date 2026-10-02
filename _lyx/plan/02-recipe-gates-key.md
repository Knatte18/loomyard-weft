# Card 2 — Recipe rows declare their gates as a `gates` list

**Edit:**
- `internal/shedrecipe/config.go`
- `internal/shedrecipe/config_test.go`
- `internal/shedrecipe/entries_gate.go`
- `internal/shedrecipe/entries_burler.go`
- `internal/shedrecipe/entries_describe.go`
- `internal/shedrecipe/entries_discussionwrite.go`
- `internal/shedrecipe/entries_planwrite.go`
- `internal/shedrecipe/entries_prrework.go`
- `internal/shedrecipe/entries_simple.go`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/shedrecipe/entries_describe_test.go`
- `internal/shedrecipe/entries_discussionwrite_test.go`
- `internal/shedrecipe/entries_planwrite_test.go`
- `internal/shedrecipe/entries_prrework_test.go`
- `internal/shedrecipe/entries_simple_test.go`
- `internal/shedrecipe/registry_test.go`
- `internal/loomrecipe/gatequiescence_test.go`
- `internal/loomrecipe/resume_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/loomcli/smoke_gate_test.go`
- `contracts/recipes/loom-recipe.yaml`

**Create:**
- `internal/shedrecipe/entries_gate_test.go`

**Prosa:**
- `contracts/specs/shed-recipe-spec.md`

**Uses:**
- `plan:internal/shuttleengine#GateEntry`
- `internal/shuttleengine#GateSpec`

**Intent:**
Replace the `gate`/`gate_attempts` row keys with one `gates` key, a list of maps (decision record Decision 8), migrate the shipped recipe, and document the key (Decision 10).

`config.go` gains a list-of-maps accessor in the style of `configStringSlice` and `configMap`: it accepts a `[]any` whose every element is a `map[string]any`, returns each as a `Config`, and names the key and the offending element's index when an element is not a map.
Unlike the other accessors, a present empty list is not absent: the accessor reports it distinctly so `resolveGateSpec` can reject it, since an absent key is ungated and an empty list is an author mistake.
`config_test.go` covers the accessor's shapes.

`resolveGateSpec` stays the single place gate config is read (Shed Recipe Registry Invariant) and reads only `gates`.
Absent key: the empty `GateSpec`, ungated.
Per element: `name` required, resolved against the existing closed vocabulary (`discussion`, `plan`, `rework-plan`, `description`) with the same `Env` requirements as today;
`attempts` required, a non-negative integer, read through `configInt` so `0` is a present value;
`pass_on_cap` optional bool, default false;
any other element key rejected through `configRejectUnknown`.
An entry with `attempts` 0 still resolves its closure and its `Env` requirements, so a typo in an off gate fails loud.
A present but empty list, a duplicate `name` within one list, a negative `attempts`, a missing `name` or `attempts`, an unknown element key and an unknown name are each an error qualified with the row's `entry`.
`configString`, `configInt`, `configBool` and `configRejectUnknown` name no entry, so `resolveGateSpec` wraps every error they return for a `gates` element with the entry and the element's index, as its own `Errorf` sites already name the entry.
Each gate-capable entry's `configRejectUnknown` list drops `gate` and `gate_attempts` and accepts `gates`, so a leftover old key fails as an unrecognized key;
that error names the key, and the row is named by `shedbuild`'s per-row wrap (`internal/shedbuild/build.go`), as for every other unrecognized key.
Update the doc comments on `resolveGateSpec` and the gate-capable entries that name the old keys or the "0 means default" rule.

`contracts/recipes/loom-recipe.yaml`: every gated row (`Discussion-Write`, `Discussion-Burler`, `Plan-Write`, `Plan-Burler`, `Describe`, `PR-Rework`) becomes
```yaml
gates:
  - name: <its current gate value>
    attempts: 3
```
with no `pass_on_cap`, so each row behaves exactly as before.
The header and row comments describing the gate (`gate_attempts` in "up to gate_attempts times") describe the list form instead.
No `gate:` or `gate_attempts:` key remains in the file.

`internal/loomrecipe/gatequiescence_test.go` detects a gated row by a non-empty `gates` list rather than a `gate` key, keeping both tests' assertions;
its doc comments and failure messages that name a `"gate"` config key name the `gates` key instead.
`internal/loomcli/smoke_test.go`'s comment naming the `gate_attempts` budget names the row's `gates` entry instead.
`internal/loomrecipe/resume_test.go`'s `TestBounceRouting_BudgetExhaustionBlocks` comment and `internal/loomcli/smoke_gate_test.go`'s comment above `loomshed.NewDiscussionGate`, which both name a `gate: discussion` row, name a `gates` entry named `discussion` instead.

Tests: the new `entries_gate_test.go` drives `resolveGateSpec` directly and covers a valid one-entry `gates` list per vocabulary name (closure present, `Name` and `Attempts` carried), `pass_on_cap` parsed, an `attempts`-0 entry resolved, an absent key ungated, and each rejection above with the row named in the error.
The existing entry tests that built `Config{"gate": ..., "gate_attempts": ...}` build the `gates` form instead and keep their assertions;
the tests of the removed keys (`gate_attempts` without `gate`, the planwrite "absent gate_attempts is the default sentinel" case, the webster `gate_attempts`-only case) are replaced by the matrix in `entries_gate_test.go` plus one leftover-key rejection per gate-capable entry, each asserting the unrecognized-key error names the old key.

`contracts/specs/shed-recipe-spec.md` gains a short paragraph beside the `commit_seam` selection rule: a row's `gates` key selects validators by name from the closed vocabulary, in order, each with its own `attempts` budget (0 means off) and optional `pass_on_cap`;
an absent key is ungated, and an empty list, a duplicate name, or a negative budget is a construction error.
Semantic line breaks throughout.

**ImpactSummary:** Changes the recipe author contract for every gate-capable row and the embedded loom recipe; runtime behaviour of every shipped row is unchanged, and the old keys are read only by `shedrecipe` and by one `loomrecipe` test (`gatequiescence_test.go`), both migrated here.

**Commit:** 2: shedrecipe: read a row's gates list in place of gate/gate_attempts
