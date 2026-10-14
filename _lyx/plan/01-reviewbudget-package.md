# Card 1 — reviewbudget package

**Create:**
- `internal/reviewbudget/doc.go`
- `internal/reviewbudget#RunSoftKey`
- `internal/reviewbudget#RunHardKey`
- `internal/reviewbudget#SoftKey`
- `internal/reviewbudget#HardKey`
- `internal/reviewbudget#AutoAcceptKey`
- `internal/reviewbudget#Categories`
- `internal/reviewbudget#Caps`
- `internal/reviewbudget#Resolve`
- `internal/reviewbudget#Values`
- `internal/reviewbudget#Route`
- `internal/reviewbudget#CheckResolved`
- `internal/reviewbudget#CheckLocal`
- `internal/reviewbudget/reviewbudget_test.go`
- `internal/reviewbudget#TestCheckResolved`
- `internal/reviewbudget#TestCheckLocal`
- `internal/reviewbudget/leaf_enforcement_test.go`
- `internal/reviewbudget#TestLeafInvariant_AllowlistOnly`
- `internal/reviewbudget/testmain_test.go`
- `internal/reviewbudget#TestMain`

**Edit:**
- `pattern/PATTERN-leaf-packages.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/configengine#StackLayers`
- `internal/configengine#MarkInvalid`

**Intent:**
D1: one declarer of the review budget key family that loom and darn config both build on, so the two recipes cannot drift apart.
Production code lives in `internal/reviewbudget/reviewbudget.go` and imports only the standard library and `internal/configengine`, the closed set the decision record's constraint fixes; it declares no `ConfigTemplate`, so the configreg registration oracle ignores it.
Nothing calls it yet: cards 4 and 5 move loom and darn onto it.

What it declares:

- `RunSoftKey` and `RunHardKey` are the run-wide keys; `SoftKey(p)`, `HardKey(p)` and `AutoAcceptKey(p)` return `<p>_max_bounces`, `<p>_hard_max_bounces` and `<p>_auto_accept`.
- `Categories(perches)` returns the category of every key of the family: the two run-wide keys and every per-perch soft and hard key `configengine.CategoryNone`, and every `<perch>_auto_accept` key `configengine.CategoryGuard`.
  With no perches it returns the two run-wide keys alone, which is darn's family.
  No skip key is declared: per the record's "Perch budgets carry no guard" and "Zero-round perches: settled design" entries, a budget of 0 is the off switch and no budget or off switch is a guard key.
- `Caps` holds `SoftKey string`, `Soft int`, `HardKey string` and `Hard int`, the shape `loomengine.PerchCaps` has today.
- `Resolve(perch, runSoft, runHard, ownSoft, ownHard)` returns the perch's caps: the soft cap is `*ownSoft` under `SoftKey(perch)` when `ownSoft` is non-nil, so an own 0 is zero rounds, else `runSoft` under `RunSoftKey`; the hard cap is `ownHard` under `HardKey(perch)` when above 0, else `runHard` under `RunHardKey`.
  A nil own soft cap is the inherit meaning the settled design moves from 0 to a null value; hard caps keep 0 as inherit, per its point (7).
- `Route(dirs, spec, key)` is `loomengine.CapRoute`'s layer switch over `configengine.StackLayers(dirs, spec)` and its `Origins`, with `spec.Name` as the module in every command:
  the local layer gives `run "lyx config <module> --layer local --set <key>=<n>" in the task worktree, and correct the task's run_settings with "lyx board upsert" too, so a later Seed-Child writes the same value`;
  the worktree layer gives `run "lyx config <module> --set <key>=<n>" in the task worktree`;
  any other origin gives, for a module whose `spec.HubWide` is false, `run "lyx config <module> --layer default --set <key>=<n>" from the hub's prime worktree, where the operator or the hub orch runs it`, and for a hub-wide module `run "lyx config <module> --set <key>=<n>" from the prime`.
  The loom strings are `CapRoute`'s today, byte for byte; the hub-wide one is the command darn's refusals name today.
- `Values` holds `RunSoft int`, `RunHard int`, `Perches []string`, `PerchSoft map[string]*int` and `PerchHard map[string]int`.
- `CheckResolved(dirs, spec, values)` refuses, in this order, each message opening `<spec.Name> config key "<key>": ` and ending `; way forward: <Route of that key>, then run "lyx loom resume"`:
  `RunSoft` below 0: `must not be negative, got <v>; set it to 0 for no review round, or to a positive integer`;
  a non-nil `PerchSoft` entry below 0: `must not be negative, got <v>; set it to 0 for no review round, to a positive integer, or to null to take review_max_bounces`;
  `RunHard` below 1: `must be at least 1, got <v>`;
  a `PerchHard` entry below 0: `must not be negative, got <v>; set it to 0 to take review_hard_max_bounces, or to a positive integer`;
  then, for each perch in `Perches` order, or for the run-wide pair alone when `Perches` is empty, a resolved hard cap below its resolved soft cap: `resolved hard cap <h> is below "<softKey>"'s resolved soft cap <s>`, ending instead `; way forward: raise the hard cap: <Route of hardKey>, or lower the soft cap: <Route of softKey>, then run "lyx loom resume"`.
  A `Route` error is returned as is.
- `CheckLocal(set, perches)` checks the family's budget keys present in `set`, a local-layer document its caller decoded into a map, and leaves every other key alone: `RunSoftKey` below 0, a per-perch soft key below 0, `RunHardKey` below 1 and a per-perch hard key below 0 are each refused with `key "<key>": must be at least <bound>, got <v>; correct the task's run_settings with "lyx board upsert"`.
  A nil value (a YAML null) passes only for a per-perch soft key (`SoftKey(p)`), the one key a null gives a meaning; a nil `RunSoftKey`, `RunHardKey` or per-perch hard key, and any budget key whose value is neither an `int` nor nil, is refused with `key "<key>": must be an integer`.
  Every refusal is marked `configengine.ErrInvalid`.

Tests, unit tier, `t.Parallel`:
`TestCheckResolved` is a table over a per-worktree spec and a hub-wide spec whose layer files the rows write under `t.TempDir()` anchor and board dirs.
It reaches each refusal above with its prefix and its route by layer: a local hard cap names `--layer local` and `lyx board upsert`, a worktree one `lyx config <module> --set` in the task worktree, a default one the prime command of each spec kind.
It reaches the hard-below-soft refusal with the soft cap from the local layer, naming both the local and the hub route, and a clean set returning nil.
Each row also asserts the `Resolve` result for its perch: a nil own soft cap resolving to the run-wide value and key, and an own 0 resolving to 0 under the perch's own key.
`TestCheckLocal` is a table: each bound refused naming key and bound, a null per-perch soft key and a run-wide 0 accepted, a non-budget key ignored, a string value and a null `review_max_bounces` each refused as not an integer, every refusal matching `configengine.ErrInvalid`.
`TestLeafInvariant_AllowlistOnly` calls `scankit.AssertImportAllowlistNoStale` on `internal/reviewbudget` with `internal/configengine` alone.
`TestMain` runs through `tmuxkit.Main` as every package's does.

`pattern/PATTERN-leaf-packages.md` gains a `## \`internal/reviewbudget\`` section: imports only the standard library and `configengine`; it declares the review budget key family loom and darn config build on, and imports neither engine.
`cmd/lyx/testdata/test-budget.yaml` gains the row `internal/reviewbudget: 3` in sort order.

**ImpactSummary:** a new leaf package with no caller yet, plus its PATTERN row and test-budget row.

**Commit:** 1: reviewbudget: one declarer of the review budget key family
