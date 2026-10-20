---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/prebuild_test.go:1
    summary: the store's race and error paths in copyIntoPlace and prebuildLyx have no test, though the focus directive names concurrency safety and the Windows rename branch
    fix: add rows to TestStorePrebuilt in internal/gatecli/prebuild_test.go. One row pre-publishes the destination entry under the content name before the call and expects the same path returned and no leftover .partial- file in the store. One row points storeDir at a regular file and expects storePrebuilt to return an error. Do not add a new test function.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/prebuild.go:84
    summary: storePrebuilt checks for an existing entry with Stat and then refreshes it with Chtimes, so a concurrent prune that removes the entry between the two fails the gate run with a keep-the-prebuilt-lyx error
    fix: in storePrebuilt, when os.Chtimes on an existing entry returns an error satisfying errors.Is(err, os.ErrNotExist), fall through to copyIntoPlace instead of returning the error.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/test_integration_test.go:538
    summary: the documented outside-a-hub behaviour, where a tagged run keeps the per-invocation build path and removes it afterwards, has no test because the tagged-run subtest runs only in a hub
    fix: in internal/gatecli/test_integration_test.go add one assertion to the tagged-run subtests, or one tagged run started from a directory outside any hub, expecting the exported prebuilt path to be the build's own -o path and its directory removed after the run.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/shedverbs/loop.go:453
    summary: the child-environment code in shedverbs still spells the LYX_TRACE_ID literal although logger.TraceIDEnv now names the variable, so a rename would miss these sites
    fix: in internal/shedverbs/loop.go lines 453 and 457 use logger.TraceIDEnv in place of the literal, provided shedverbs already imports internal/logger and PATTERN-leaf-packages allows it; otherwise leave the literal.
---

### [MEDIUM] Store race and error paths untested

**Location:** internal/gatecli/prebuild_test.go:1
**Issue:** `TestStorePrebuilt` covers the name, reuse with a refreshed time and pruning. The rename-lost-race branch of `copyIntoPlace`, the staging cleanup on a failed copy and the wrapped store failure in `prebuildLyx` are new behaviour with no row. The focus directive asks specifically about concurrency and the Windows rename.

### [LOW] Chtimes on an entry pruned by a concurrent run

**Location:** internal/gatecli/prebuild.go:84
**Issue:** The window is microseconds and the entry must already be past the two-hour retention, so the failure is rare. It still fails a whole verify run on a state the next call would repair.

### [LOW] Non-hub tagged path untested

**Location:** internal/gatecli/test_integration_test.go:538
**Issue:** `prebuildLyx` with an empty `storeDir` is the documented fallback and changed signature in this diff. Only the hub path is exercised.

### [NIT] Trace id literal outside the logger

**Location:** internal/shedverbs/loop.go:453
**Issue:** `logger.TraceIDEnv` is the only name used in the gate and logger code. `loop.go` and its test use the literal for the child's environment, which the new constant does not reach.

## Rejected

- Staging file left by a killed run whose pid is reused (code-correctness LOW): the failed `O_EXCL` create removes the file, the next run succeeds and the file is pruned after two hours anyway.
- No fsync before the rename (code-correctness LOW): speculative crash-consistency case; a crash while a gate run is in flight is retried by the run itself.
- Plan-verify record gate assembly pinned only at step level (code-tests LOW): the assembly branch is shared with the `publish_verify` integration cases, which pin the order.
- `PrebuiltDir` compared with itself in the integration test (code-tests NIT): a rename of the directory is a deliberate edit of a one-line accessor with its own doc line.
- Stored content and prefix re-asserted in the integration subtest (code-tests NIT): the subtest checks the wiring from the build to the exported path, which the unit test does not.

Holistic checks that held: the diff makes each of the four asks of the board entry and nothing it does not license, the trace id is dropped in every branch, the failing-test steps run before the told or derived command and the package pass after it in both forms, `publishFailureSteps` has one caller, and the docs, `PATTERN-test-speed.md` and `CLAUDE.md` match the behaviour.
