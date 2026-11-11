# Coverage gate scope: changed packages whole, importers' selected tests

A gate key may now hold `scope: coverage`.
Under the `go` scoper it takes the `diff` scope and narrows each package that is in the set only because it imports a changed package.
Such a package runs only the tests its coverage map says the change can affect.
Changed packages still run whole.
The template's `webster-fix`, `darn-write` and `code-burler-fix` keys default to it, and `pr-review` stays on `diff`.

Why: tiers 0 and 1 carry most of the verify time.
An importing package whose linked code changed in one function need not rerun every test.
The landing gate, of scope `all`, still runs every test before a landing.

## What changes

- **`internal/gateslot`**: `ScopeCoverage` is a new scope value.
  The loader accepts it, and an unknown scope's error names `coverage` among the allowed values.
  It is a new value rather than a changed meaning of `diff`, so a hub file holding `diff` keeps its meaning and needs no reinterpretation.
  A binary that predates the value refuses a hub file holding it until it updates.
  The template, its comment on the gate keys, the package doc and `PATTERN-test-speed`'s key table name the new defaults.
- **`internal/impactset`**: `ModuleTargets` gains `Importers` and `BinaryBuilders`.
  Importers are the packages the set holds only because their build or tests reach a changed package.
  Binary builders are those it holds because `cmd/lym` changed and their tests build the `lym` binary.
  The rest of `Packages` are the changed packages.
  `Packages` itself is unchanged, so every existing caller is unaffected.
- **`internal/testcover`**: `Selection` gains `Recorded`, the tests the map records.
  This lets the gate find the tests the map has no coverage for from the same read that made the selection.
- **`internal/gatecli`**:
  - `withGateScope` treats a coverage key as the computed form and marks the request.
  - `computeScope` derives each base group exactly as under `diff` (bases, fallbacks, inert paths, guard and watched tests).
    It then calls `narrowCoverage` for each tier of a narrowed share.
  - A changed package and a binary builder run every test of the tier.
  - An importing package runs, per tier, the union of these, each a Test function of the tier declared in a test file the host admits under the tier's tag:
    - the tests its map selects;
    - the tests the map does not record;
    - the tests the map marks out of process or unresolved;
    - its `//lym:guard` tests.
  - The watch skip drops tests from the union.
    A selector miss runs the package whole, and an empty union runs nothing.
  - `planTierRuns` adds a second run of scope form `selected` over the selected packages, after the tier's own run.
    Its args are the call's own words followed by `-run '^(<names>)$'`.
    A pattern past `greenCommandBudget` runs the selected packages whole in the first run instead.
  - Because the filter sits in the run's args:
    - the green entry carries it, so the entry never becomes a diff base and never stands in for an unfiltered run;
    - the empty-tier check, which reads only the call's args, still checks the run.
  - The selected run is labelled by its tier, so `--stop-on-fail`, the failed-test rerun's parse of the failure report and the closing line treat it as any tier run.
    `tierNumbers` lists a tier once even when it has both runs.
  - Stderr names each package of a narrowed tier as `gate test: tierN coverage: <package> <label>`.
    The label is one of: `changed`, `whole: lym binary`, `miss: <reason>: <detail>`, `whole: over budget`, `selected <n> of <m>` or `none selected`.
  - The maps are read from the hub's `testcover.Dir` through `testcover.Selector`, which `ReadMap`s them.
    No scope computation starts a build or a refresh.

## Design decisions

- The narrowing is skipped in these cases, and the share then runs as under `diff`:
  - the call's words after `--` carry a filter flag, since a second `-run` would fight the caller's own;
  - the tier's command lacks `{args}` or a target placeholder;
  - the tier has `carry: false` and so runs whole anyway.
- The selector opens lazily, once per call, and caches each package's selection across tiers and base groups.
  It reads the working files as HEAD's tree plus the uncommitted and untracked paths through `fabricengine`.
  It reads the toolchain root with `go env GOROOT` from the call's go binary.
  A failure in any of these makes every importing package miss, so the package runs whole.
  Outside a hub there are no maps, and every importing package misses as `no coverage map`.
- When `cmd/lym` changes, a package both reached through imports and building the binary counts as a binary builder and runs whole.
  The coverage map sees only in-process code, so it cannot speak for the binary.
- A test file in an importing package that cannot be read or parsed runs the package whole, with a `Go file does not parse` miss.
- `Selection.Recorded` is a small addition to `internal/testcover`, which lies outside the three packages the record names.
  The alternative was a second `ReadMap` that could see a different map than the selection did.

## To check by hand

- The new tests are tier 0 and cover:
  - the union rules and labels in `narrowCoverage`;
  - the two-run plan;
  - the coverage key's request marking and the tiers it narrows;
  - the importer split in `targetsFor`;
  - the loader's new value.
  No integration test drives a whole `lym gate test --gate darn-write` over real coverage maps.
  The selector's own reads, the green entry of a real selected pass, and the rerun and `--stop-on-fail` behaviour over the second run are covered only by the existing tests of those mechanisms.
- No test was run in this session; the verify gate runs them on the committed tree.
