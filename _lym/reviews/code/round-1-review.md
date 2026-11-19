---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/stencilstore/skew.go#shippedOverNewer
    summary: Read and readInclude now render a prompt from the embedded registry bytes, which PATTERN-stencil-ownership forbids for every role but the orch, and the range carries no Pattern-Departure trailer or PATTERN change.
    fix: In internal/stencilstore/skew.go, replace the fallback with the Sketch's other option, an early refusal. Have contentForRunning return an error instead of substitute bytes when shippedOverNewer's conditions hold (registered registry, running commit time set, recorded writer newer, body differs from the shipped default). Name the error after the skew, and have it name the stencil, the writer's revision, the running revision and the way forward, restarting the run's driver on the current lym binary. Propagate that error from Read (reconcile.go) and from readInclude's engine-block path (engineblock.go). Keep the registry registration only to compare bodies, never to return them. Drop newerCopyWarned and the Warn log. Rewrite the Read doc comment, the new doc.go paragraph and the skew.go file header and symbol docs to describe the refusal, and remove the claim that the process renders its own shipped default.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/stencilstore/skew_test.go#TestShippedOverNewer
    summary: The skew case is tested only through the unexported shippedOverNewer, against PATTERN-test-speed's rule that an unexported helper is tested through the public surface. No test reaches the wiring through Read, readInclude or the registration, nor the equal-time and no-writer boundaries.
    fix: In internal/stencilstore, add an export_test.go that sets the shipped registry and the running writer for a test, and name the process-global state in a comment on the test that uses it, which cannot call t.Parallel. Replace TestShippedOverNewer with a table-driven test of the public read path over a t.TempDir() board. Its rows hold a newer-writer untouched copy of a top-level stencil read by Read, of a plain include read by ReadForEngine, and of an engine block read by ReadSet, each asserting the skew error and its wording parts from F1's fix. Rows for an older writer, an equal commit time, a copy with no recorded writer, a newer copy with the shipped body, a running binary with no commit time and an unshipped name each assert the board bytes. Every row asserts the board file is unchanged afterwards.
  - id: F3
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/stencilstore/skew.go#shippedOverNewer
    summary: A board copy whose recorded writer is newer but whose body was edited after that write is read as it stands with no skew detection, so a newer body that adds an include or marker still reaches the old renderer and fails with the bare template error the Sketch asks to replace.
    fix: In shippedOverNewer, or its replacement under F1's fix, apply the newer-writer refusal whether Classify returns StateUntouched or StateEdited, as long as the body differs from the shipped default. Never write or replace the board file. In F2's test, change the EditedCopyFromANewerBinary expectation to the skew error.
  - id: F4
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/stencilstore/skew.go#shippedOverNewer
    summary: The read path orders writers by commit time alone, while the production refresh orders them by ancestry first (fabricengine.StencilSource's Older). A descendant binary with an equal or earlier commit time can refresh a board copy that the running ancestor then reads unchecked.
    fix: Let the caller that registers the shipped registry also register the ordering the refresh uses. In cmd/lym's startup, or wherever the process builds its stencilstore.Source, pass Source.Older from fabricengine.StencilSource to stencilstore alongside the registry. Treat a recorded writer as newer when that ordering answers RecordedNotOlder and its revision differs from the running one. Fall back to the commit-time comparison only when no ordering is registered. Add rows to F2's test for a descendant writer with an equal and with an earlier commit time, using a fake ordering.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/stencils.go:1134
    summary: The Registry doc comment says stencilstore.Read needs no registry, but Read now depends on the registry this package's init registers.
    fix: In contracts/stencils/stencils.go, replace the sentence An engine reads a stencil at call time via stencilstore.Read, which needs no registry. with one stating that init registers this registry with stencilstore, so a read can detect a board copy that a newer binary wrote.
---

### [BLOCKING] The fallback renders prompts from embedded bytes, against PATTERN-stencil-ownership

**Location:** internal/stencilstore/skew.go#shippedOverNewer

**Issue:** `pattern/PATTERN-stencil-ownership.md` rules that prompts are read "at call time via `stencilstore`, never embedded, bar the orch role", and that "None is read from embedded bytes".
`shippedOverNewer` returns `registry.Default(name)`, the `//go:embed` bytes `contracts/stencils` registers at init, and `Read` and `readInclude` hand those bytes to every renderer.
Neither commit in the range carries a `Pattern-Departure:` trailer, and the directive declares no PATTERN change.
The Sketch allows two ways: an early error naming the skew and the way forward, or a fallback where that is safe.
Where a constraint conflicts with the Sketch the constraint wins, so the fallback branch is closed and the error branch remains open to the fix.
The affected-set lens and the assistant reached the same finding independently.

### [BLOCKING] The skew behaviour is tested only through an unexported helper

**Location:** internal/stencilstore/skew_test.go#TestShippedOverNewer

**Issue:** `pattern/PATTERN-test-speed.md` states that a test targets behaviour at the module's public surface, and that an unexported helper is tested through the surface that uses it.
`TestShippedOverNewer` calls `shippedOverNewer` directly.
It stays green when the `contentForRunning` call is removed from `Read` or from `readInclude`, when the engine-block key is wrong, or when the `init` registration is dropped.
`_lym/landing/summary.md` concedes that the wiring "is not exercised by a test".
The table also has no row for an equal commit time or a copy with no recorded writer.
So flipping `!recorded.Time.After(running.Time)` to `recorded.Time.Before(running.Time)` passes every row, against the doc's promise that a writer with no time is never newer.

### [MEDIUM] An edited copy from a newer writer bypasses skew detection

**Location:** internal/stencilstore/skew.go#shippedOverNewer

**Issue:** The `Classify(...) != StateUntouched` guard returns any edited board copy unchanged, whatever its recorded writer.
Suppose an operator appends a line to a newer binary's `gatefix-body-darn` copy, the one that adds `{{template "loom-fragment-loosened-check"}}`.
The copy keeps its newer writer, classifies as `StateEdited`, and reaches the old binary's renderer, which fails with the same bare template error as issue #149.
Preserving an operator's edit means leaving the file alone; it does not require feeding the incompatible text to the renderer.
The Sketch asks for the skew to be detected and refused with a way forward where a fallback is not safe.

### [MEDIUM] Writer ordering on the read path disagrees with the refresh's ordering

**Location:** internal/stencilstore/skew.go#shippedOverNewer

**Issue:** `fabricengine.StencilSource` (`internal/fabricengine/warpforward.go:65-104`) answers `RecordedOlder` by strict ancestry when the worktree holds both commits, and by commit time only otherwise.
A production refresh by a descendant binary whose commit time equals or precedes the running binary's therefore rewrites board copies.
The running ancestor's `!recorded.Time.After(running.Time)` check then lets that newer body through unchecked, and the bare template error recurs.
The correctness lens raised the same gap at LOW, for a branch build with a later commit time; it is merged here.

### [LOW] Registry doc says Read needs no registry

**Location:** contracts/stencils/stencils.go:1134

**Issue:** The `Registry` doc comment says an engine reads a stencil "via stencilstore.Read, which needs no registry".
After this diff, `Read` consults the registry the package's own `init` registers through `RegisterShipped`.
That holds under F1's fix as well, since the refusal still compares bodies against the registry.

## Focus results

- `Classify(onDisk, true, def)` returns `StateUntouched` only when a parseable stamp equals the body hash (`internal/stencilstore/stencilstore.go:315-329`).
  A copy with no writer keys parses to the zero `Writer`, whose zero time is never `After`, so it cannot fall back.
- Every render read goes through `contentForRunning`: `Read`, plain includes through `Read`, and engine blocks in `readInclude`.
  The other `os.ReadFile` sites (`Validate`, `reconcileAll`, `pendingWrites`, `warnPortBackDrift`, `readBoardCopy`) inspect or reconcile board state and render nothing.
  Board paths handed to agents as text, such as `rubric_path`, are read by the agent and are not template renders.
- `EngineBlockName(engine, block)` yields `engine/<engine>/<block>`, the key the `contracts/stencils` entries carry.
- `RegisterShipped` has one caller, a package `init`, which runs once per binary and happens-before `main` and every test; no race and no second call.
- `runningWriter` is a `sync.OnceValue`, and an unstamped binary's zero time disables the path; `LoadOrStore` logs once under concurrent reads.
  F2 records that none of this wiring is tested.
- Safety of the fallback for edited, equal-time and unrelated-branch writers: F3 and F4.
- Doc comments: `doc.go`, `Read` and `skew.go` match the code as written, and F1's fix rewrites them; F5 covers the stale `Registry` doc.

## Rejected

- code-tests LOW, a test for `RegisterShipped`'s second-call panic: the panic guards misuse of an internal API whose one caller is a package `init`, which the trust-the-caller rule leaves untested.
- code-tests LOW, the missing equal-time and no-writer rows: folded into F2.
- code-correctness LOW, commit-time ordering misses a later-time branch build: merged into F4.
- code-affected-set F2 and F3, the comments at `internal/shedadapters/bouncer.go:330`, `internal/editdirective/doc.go:43` and `internal/twilldirective/doc.go:51` saying a read never falls back to embedded bytes: these hold again once F1's fix removes the fallback, so they need no change of their own.
- code-affected-set F5, the stencil-pins notice overstating which prompts change: it holds again once F1's fix removes the fallback.
- code-scope: no finding was raised.
  Its trace of G1 and G2 to the parent's added entry, with 72c3e192 an ancestor of the base, is accepted.
