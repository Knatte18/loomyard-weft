# Card 11 — entrymerge-package

**Create:**
- `internal/entrymerge/doc.go`
- `internal/entrymerge#Field`
- `internal/entrymerge#Table`
- `internal/entrymerge#Finding`
- `internal/entrymerge#Table.Changed`
- `internal/entrymerge#Table.Change`
- `internal/entrymerge#Table.AddClash`
- `internal/entrymerge#Table.Same`
- `internal/entrymerge#Render`
- `internal/entrymerge_test#TestTable`
- `internal/entrymerge_test#TestMain`

**Intent:** Draw the board replay's three-way by-entry merge out into a package of its own, parameterised by the record's key and field table (D8), so the board and heddle share it rather than copy it.

`internal/entrymerge` imports only the standard library (`encoding/json`, `fmt`, `reflect`); `doc.go` states the package's contract below and that it holds no record's own checks.

- `Field[T]` is one comparable field: `Name`, its JSON name; `Get func(T) any`, a canonical getter that reads nil and empty alike; `Set func(dst *T, src T)`, which copies it from another entry.
- `Table[T]` holds `Key func(T) string` and `Fields []Field[T]`, in the order a change reports them.
- `Finding` is `Entry`, `Field`, `Reason` and `Local`, the shape the board replay reports a loss in.
- `Changed(before, commit)` returns the fields whose values differ between the commit's parent and the commit, by `reflect.DeepEqual` over `Get`.
- `Change(base, before, origin, commit)` starts `merged` from `origin` and, for each changed field: skips it when origin already holds the commit's value; reports it lost with reason `origin changed the field too` and `Local` the commit's value through `Render` when origin's value is neither the merge base's nor the parent's; otherwise sets it from the commit and adds its name to `applied`.
  This is exactly `boardengine`'s `replayRun.changeEntry` field loop, so the board's reasons and values stay byte-identical.
- `AddClash(origin, commit)` reports each field the commit's added entry carries differently from an entry origin added under the same key, reason `origin added an entry with this slug too`, as `replayRun.addEntry` does.
- `Same(a, b)` reports whether every field is equal, the test `replayRun.removeEntry` applies before removing an entry origin still holds unchanged.
- `Render` renders a value as JSON, falling back to `fmt.Sprint`, as `replayValue` does.
- Unknown keys, ids, handles and dates are not fields: a caller merges the table's fields and keeps everything else as origin holds it.

`TestMain` lives in an untagged `internal/entrymerge/testmain_test.go` and runs the tests under `tmuxkit.Main` (PATTERN-test-isolation); the package spawns no git, so it calls no `gitkit.HermeticGitEnv()`.

`TestTable` (tier0, in a new `internal/entrymerge/entrymerge_test.go` with `//go:build tier0`) drives a two-field test record through the rows no existing test covers: an origin value equal to the parent's counts as unchanged by origin, so the commit's value is applied; `Changed` lists only differing fields and treats a nil and an empty slice alike through the getter; `AddClash` reports each differing field.
A field only the commit changed being applied, a field both sides changed keeping origin's value and being reported lost, a field origin already holds being neither applied nor lost, and `Same` failing on one differing field are covered by `boardengine`'s `TestFabricReplay`, which card 12 runs over this merge unchanged.

