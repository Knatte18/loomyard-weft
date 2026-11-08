```yaml
lens: code-correctness
outcome: reported
```

Task: judge the correctness of the code the diff changes, case by case.

**List (function or branch: case run: verdict)**

- `ModuleMainPackages`, root module with `main.go` and `cmd/tool`: returns `[".", "./cmd/tool"]`, sorted. Correct.
- `ModuleMainPackages`, module with no main package: `go list` prints records for the library packages, none has name `main`, so it returns a nil slice and no error. Correct.
- `ModuleMainPackages`, directory without `go.mod`: `go list` exits non-zero and the `ExitError` branch wraps `moduleDir` and stderr. Correct.
- `ModuleMainPackages`, record parsing: each record starts with a `\n` left from the previous record and the code trims it. A list error containing tabs stays whole through `SplitN(…, 3)`. A multi-line error does not break the record split, because records end at NUL. Correct.
- `ModuleMainPackages`, `dir == ""` (an entry from `-e` with no directory): returns an error carrying the list error text. Correct, though the message is only as informative as `go list`'s error.
- `ModuleMainPackages`, package directory outside the module, or `filepath.Rel` failing: refused. Correct, and the error names `absolute`.
- `ModuleMainPackages`, a main package that fails to build or has a file error: `-e` keeps it listed with its name, so it is run and the gate fails there. Correct. A main package whose files are all excluded by build tags may list with an empty name and be skipped. That is acceptable, since no tag is passed.
- `ModuleMainPackages`, nested modules: `./...` stops at a nested `go.mod`, so nested modules' mains are not covered. Within the Sketch ("every package main of the module").
- `ModuleMainPackages`, `ctx` cancelled: `CommandContext` kills the run and the error returns. The spawn and exit are logged, as the child-process rule requires. Correct.
- `computeScope`, `--main-packages` with a go scoper: `moduleKey(root, tier0.Dir)` gives `"."` for the root module, matching `ModulePackages.Dir` (`"."` for the root, `planparser/targetdirs.go:80`). `withMainPackages` therefore merges into the card's root-module entry. Correct.
- `computeScope`, a card scope that has no module entry (a docs-only card): `withMainPackages` appends a new module with the mains, so the mains run. Correct and intended.
- `computeScope`, mains empty: the scope is returned unchanged, with no error. Correct.
- `computeScope`, `go list` fails: returns an error with a way forward and mutates nothing. Correct.
- `computeScope`, `dirs` for the watch-skip: now built from `modules` after the merge, so the main packages are included. The `os.Stat` guard also covers `.` for a root main. Correct.
- `withMainPackages`, a package already named by a card: not added twice. `append` on `modules[index].targets` could alias a slice owned by `cardModules`. The slice is not reused afterwards, so this is harmless.
- `withMainPackages`, a card naming `./cmd/lym/...` while the mains list `./cmd/lym`: both are kept, so `cmd/lym` is run twice. This costs time only, with no wrong result.
- `computeScope`, a scoper other than `go`: returns earlier (`dotnet`, `none`, or `whole`), so the flag adds nothing. Correct, as the Sketch requires.
- `flagShapeRefusal`, `--main-packages` with no `--card`: refused with a way forward naming `--card <id>`. It sits after the more-than-one-scope-form check, and a call that has both `--all` and `--main-packages` gets the scope-form refusal first. That is fine.
- `flagShapeRefusal`, `--main-packages` with a `--gate` whose template scope is not card: `withGateScope` may rewrite the scope, in which case the flag is silently inert. The Sketch allows this. Not a defect.
- `testRequest.words` in `detachfront.go`: appends `--main-packages` once, after the `--card` words. The detached runner repeats the scope. Correct.
- `AgentGateCallWithMainPackages`: the `renderGateCall` error is returned unchanged and the flag is appended last. Correct. A `Gate` key that cannot run is refused with the same error as `AgentGateCall`.
- `isLastBatch`, empty `batches`: false. Correct.
- `isLastBatch`, normal case: it compares `batchIdentity` numbers (the first card's number). The numbers are unique per batch, and `batches` is in execution order, as the `RecoverDeps.Batches` doc (`recoverbatch.go:61`) says. Correct.
- `isLastBatch`, a batch with no cards: both numbers are 0 and the batch counts as last only if the last batch is also empty. An empty batch cannot occur. Correct.
- `renderBatchGate`, `last` true or false: picks the matching renderer. Both refuse a missing `batch` list with the config's way forward. Correct.
- `BeginBatch` and `recoverSpawn`: both call `isLastBatch(deps.Batches, batch)` with the same helper, so the begin and recovery paths agree on which batch is last. Correct.

**Findings**

None at BLOCKING or MEDIUM. One LOW observation, with no change required beyond optional dedup:

- LOW / design / `internal/gatecli/scope.go:283-297` (`withMainPackages`): dedup is by exact target string only. A card that names `./cmd/lym/...` alongside the derived `./cmd/lym` leaves both in the scope, so that package's tests run twice. Evidence: `scope.go:290-294`, where the check is `slices.Contains`. This costs time only. The fix is optional: skip a main whose path is already covered by a card target ending in `/...`.

**Tool calls**
1. Read `internal/gatecli/scope.go:186-245`: the manifest held only the diff hunks, and I needed the surrounding `computeScope` flow (`dirs`, `tagTiers`, early returns).
2. Grepped `ModulePackages` and `cardPackages` in `planparser` and `gatecli`: to confirm `Dir` is `"."` for the root module, which the `withMainPackages` key match depends on.
