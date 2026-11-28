```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed functions, branches and error paths.

**Per function or branch: the case I ran, and whether it is correct**

- `computeScope`, `ownOnly` true (`--gate card --card 1`, root module, card package `./lib`): targets become a clone of `./lib` and `cardDependents` is never called, so no `go list` runs. `guardStep` is false, the stderr line prints and `ownPackageTiers` returns nil for a non-batch key. **Correct.**
- `computeScope`, `ownOnly` true with a nested module (root `./lib`, nested `.`): there is one `moduleTargets` per module with its own packages, which matches the test's expectation (`./lib`, `.`, `./lib`, `.`). **Correct.**
- `computeScope`, `ownOnly` true with `--main-packages`: `withMainPackages` is applied to both `modules` and `own`, so the mains are listed once. **Correct.**
- `computeScope`, `ownOnly` true with zero card modules (a card whose targets hold no Go package, which `CardPackages` drops at `internal/planparser/targetdirs.go:102`): modules are empty, so the listed tiers run nothing and no guard step runs. The card gate then runs only the lints. This follows from the requested move of the guard step to the batch gate. **Correct by the fasit; not a finding.**
- `computeScope`, `ownOnly` true with a card naming a package it has yet to create: `existingDirs` drops it for the watch scan, and `tagTiers` skips unreadable directories. **Correct.**
- `computeScope`, `ownOnly` true with `--guards`: `run.go:122` runs `scanGuards` with `listedPackages` built from the own-only shares. Every guard test outside the own packages, and outside a listed tier, is left out of the listed runs and goes to the guard step (`tiers.go:206`). **Correct.**
- `computeScope`, `ownOnly` true, vet step: `tagTiers` runs over the own modules, and `tiers.go:193` plans vet runs for `scopeCard` whatever `guardStep` is. **Correct.**
- `computeScope`, `ownOnly` true, watch skip: `sc.skip` comes from the own dirs. The guard scan is skipped, so `changed` is unused. **Correct.**
- `computeScope`, `ownOnly` false (no key, or `batch`): the dependent and guard behaviour is unchanged. `cardDependents` failure path: it returns `module.Packages` and logs. **Correct.**
- `computeScope`, `ownOnly` true under the none or dotnet scoper: the call returns before this branch (`scope.go:220-227`), so nothing changes. **Correct.**
- `SlotPriority`, `decodeGates`, `goReadGates`, `batchGateCalls`/`batchGateSpec`, `CardGateCommand`, `cardFixerOn`, `loadGateConfig`: the constant replacements keep the same values, `"card"` and `"batch"`. **Correct.**
- `gatefix-body-card.md`, the single-test re-run `lym gate test --tier &lt;t&gt; &lt;pkg&gt; -- -run '^(&lt;name&gt;)$'` with a failed subtest `TestX/case`: **incorrect.** The run passes having run nothing (finding 1).
- `gatefix-body-card.md`, the same re-run for a test of a nested module, where the report's package is an import path: no `-C` is given, so the call runs in the tier's root dir and the go command refuses the package. It fails loudly, but the fixer cannot verify its fix (finding 2).
- `gatefix-body-card.md`, re-running "any other failed step by the command the report names" for a build, vet, package panic or timeout, or a TestMain failure: the report names no command for these. **Incorrect** (finding 3).

**Findings**

1. **MEDIUM**, design. `contracts/stencils/gatefix/gatefix-body-card.md:28`: the prescribed `-run '^(&lt;its name&gt;)$'` passes while running nothing when the failed test is a subtest.
   - The report's test name holds the subtest path (`internal/gatecli/counters.go:89`, "subtests included").
   - Go's `splitRegexp` does not split on a `/` inside parentheses (`$GOROOT/src/testing/match.go:181-197`), so `^(TestX/case)$` is matched against the top-level name `TestX`, nothing matches, and `go test` exits 0.
   - `emptyTierFailure` exempts any call carrying a `-run` filter (`internal/gatecli/tierrun.go:756,770`), so the gate reports a pass.
   - The fixer is told never to run the whole call, so it takes a vacuous green as a verified fix. This hits table-driven tests, which are common here (for example `TestGateTest_Scopes`).
   - Fix: give the form per level, such as `-run '^TestX$/^case$'`, or have the fixer check that the run's summary counts the test.

2. **LOW**, design. `contracts/stencils/gatefix/gatefix-body-card.md:28`: the re-run form has no `-C &lt;module dir&gt;`, so a failed test in a nested module cannot be re-run as written.
   - Explicit targets run in the `-C` dir, else the tier's dir (`internal/gatecli/tiers.go:278`).
   - The report names the package by import path (`internal/gatecli/counters.go:87`) and the run's module dir only in its qualified file paths (`internal/gatecli/failurereport.go:302-327`).
   - The red card call ran each nested module in its own directory (`scope.go:248-254`, `doc.go:161`).

3. **MEDIUM**, design. `contracts/stencils/gatefix/gatefix-body-card.md:28`: "any other failed step by the command the report names for it" points at something the report does not hold.
   - `reportLines` prints `label (outcome)`, an optional way forward and the log path for every failed run (`internal/gatecli/failurereport.go:236-272`).
   - The one command it ever names is `lym loom lint-&lt;x&gt;`, and only in the cut line of a truncated lint step (`failurereport.go:285-286`).
   - A build failure (`buildFailedRun`, `failurereport.go:91-106`), a `vet tierN` run, a package failure outside any test, and a guard run all carry no command.
   - Line 29 bars the whole call, so the fixer has no sanctioned way to re-run those checks.

**Tool calls**

- `grep CardPackages` in `internal/planparser` together with `testing/match.go` `splitRegexp`: the manifest held neither the card-package derivation for non-Go targets nor Go's `-run` splitting rule, which decide the zero-module case and finding 1.
