```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, I traced what calls, reads, asserts or documents it, and checked whether the diff leaves any of those wrong.

**Per change, the set found and whether it is covered**

1. **`AddOptions.BackoffSleep` now also paces the origin probes** (`internal/fabricengine/add.go:42-46`).
   - Callers of `Add` that leave it nil sleep for real, which is the intended production behaviour:
     - `battencli/wire.go:1195` (Worktree-Create)
     - `battencli/readopt.go:82` (re-adoption)
     - `fabriccli/fabric.go:1116`
     - `adoptcli/deps.go:77`
     - `hubforge/hub.go:477`
   - `syncOptions` passes the field on unchanged (`add.go:107`).
   - Covered.
2. **`retryOriginProbeInTransit` and `originProbeBackoffDelays`**
   - It follows the same shape as `retryTransportPush` (`pushclassified.go:205-222`).
   - The delays are a package-level slice, but the sleep is injected through a struct field, so no test waits them out. That keeps PATTERN-test-speed's "Time is injectable" rule.
   - The longest wait is about 7 seconds, which makes it a bounded wait outside PATTERN-daemon-wakeups.
   - The probe fetches use `--refmap=` and so take no remote-refs lock, as PATTERN-gitrepo-client-boundary allows.
   - Covered.
3. **Add now retries for every caller, not only batten.**
   - I checked every existing test that points origin somewhere broken before calling Add:
     - `add_rollback_adopt_test.go:270` uses a missing path.
     - `add_remoteleftover_integration_test.go:599` and `:640` use a missing path.
     - `lifecycle_integration_test.go:832` uses a missing path.
     - `adoptremote_integration_test.go:190` and `:267` use a missing path.
   - A missing path gives "does not appear to be a git repository", which is not a transport failure. None of these tests now waits out a real backoff, and their assertions still hold.
   - The `127.0.0.1:1` tests in `battencli/catchup_integration_test.go` and `landpull_integration_test.go` never reach Add.
   - `readoptPermanent` (`readoptstop.go:28`) still reads a transport failure as transient, which fits.
   - Covered.
4. **Add's doc comment and the test file header** gained the retry sentences (`add.go:170-171`, `add_remoteleftover_integration_test.go:5`).
   - `battenshed/create.go:40-41` says the hold covers whatever probe time is left. That is still true, though a retry now makes that time longer.
   - `fabriccli/fabric.go:1113` says "origin probes always on", which is unchanged.
   - Covered.
5. **`battenTacklerRecipe` and `battenArbiterRecipe` text**
   - `driverdeps_test.go:114` compares against the constant itself, so it stays correct.
   - Both reach the prompt through the `{{.recipe}}` marker (`contracts/stencils/shed/shed-template-tackler.md:8,36` for the tackler, `shedarbiter/prompt.go:89` for the arbiter).
   - The tackler stencil's body says it is recipe-blind and that run detail arrives through the marker, so putting batten-specific guidance in the recipe text fits it.
   - The text is still a single line, so the "one-line description" doc comments (`drive.go:49`, `tacklerprompt.go:27`, `tackler.go:68`) still hold.
   - No stencil prose was changed.
   - Covered.
6. **`primeShuttle` skips the usage recorder when the task worktree is absent** (`drive.go:533-546`).
   - Its callers are `loadedTacklerDeps` and `loadedArbiterDeps`.
   - `TestPrimeShuttle_WiresTheReplyProbe` (`wire_seams_test.go:302`) runs with no task worktree and asserts only the reply probe, so it still passes.
   - `absentTaskWorktreeRefusal` keeps its wording and its other callers. Every one of those runs past Worktree-Create, as the summary says.
   - Covered.
7. **Commit messages and the repository's rules**
   - `e1ecf92` carries a `Root-Cause:` trailer that names the defect the diff removes: the first probe error was returned with no retry.
   - The range has no `Pattern-Departure:` trailer, needs no `Loosened-Check:` because no assertion was loosened, and declares no PATTERN changes.
   - Covered.

**Findings:** none.

Tool calls:
1. A grep of docs, callers and `AddOptions{` sites, to find the set outside the files the manifest had read.
2. A grep of `fabriccli/fabric.go` and `battenshed/create.go` for probe docs and of battencli for transport classification, to check docs that describe the probes.
3. A grep of the stencils for the `recipe` marker. It found only Go files, because the stencils live under `contracts/`.
4. A grep of `contracts/` for the tackler stencil's `{{.recipe}}` marker, to see how the changed recipe text is placed in the prompt.
5. A grep for tests that point origin at an unreachable URL and then call Add, to check none now waits out the real backoff.
6. `sed` of `add_rollback_adopt_test.go:262-300`, to confirm which kind of broken origin its probe-failure test uses.
