```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each Decision's mechanism end to end on one concrete case.

**Walkthrough list**

- **D1** — `cmd/testtiming -redundancy` calls the engine on `internal/fabricengine`. The engine's runner, eligibility and `classify` give the same verdicts. Purpose met.
- **D2** — a fabricengine test covers its own blocks plus fixture-package blocks. The module-wide profile names every instrumented block, so a filter on the package's own files gives the package view. Purpose met.
- **D3** — a profile line `x/a.go:10.5,12.3 1 1` is keyed from the file on disk. D5 builds only clean packages, and D7 keys from the current files, so profile and text always agree. Purpose met.
- **D4** — the map is written after a D5 build: the key table, then each test's covered block indexes. Purpose met.
- **D5** — prime, after a landing, with `--max 5` and an untracked `.go` file in an early package. Purpose not met: the package is stale and skipped on every call (finding 2).
- **D6** — 30 packages go stale after a prime HEAD move. Batches of 5 drain them in 6 ticks, then the loop in finding 2 takes over. Purpose met only for clean packages.
- **D7** — the `webster` key in a run worktree, one new test in `internal/x`. Step 1's input does not exist (finding 1). On this hub the step never runs at all (finding 3). Steps 2 to 5 hold, given step 1's output.
- **D8** — webster end gate writes R1, Code-Burler-Review reads it, Code-Burler-Fix commits, the `code-burler-fix` gate writes R2, and the next review reads R2. Hashes match at each read. Purpose met, but only once D7 runs.
- **D9** — review-text change, no mechanism to walk. Met.

**Findings**

1. **MEDIUM — design — D7 step 1**
   - Summary: D7 step 1 needs each changed test's function text at the merge base, but "the gate's existing diff route" yields changed paths only.
   - Evidence: `internal/gatecli/scope.go:476-494` (`cardChanges` → `impactset.ChangedPaths`). `internal/impactset/targets.go:102-105` documents that it returns only "every path that differs between base and the working tree".
   - Why it breaks: no file content at the base comes back, so the step cannot compare function text. The record never names how base-side text is read. Under `PATTERN-fabric-boundary` that read has to go through fabricengine, and the only file-at-revision reader there, `fabricengine.AnchoredFileAt` (`internal/fabricengine/trailercommit.go:149-151`), reads the weft commit anchor-relative.
   - Rule this breaks: the rubric's deferred-research rule.

2. **MEDIUM — design — D5 and D6**
   - Summary: a package that stays dirty in the prime re-triggers a refresh on every dobby tick and can stall progress.
   - The two rules involved:
     - D5 marks a package stale when an untracked or changed Go file differs from the map hash, and skips it when a file differs from HEAD. That package is stale and skipped on every call.
     - D5 then reports it among the stale packages left unbuilt, and D6 restarts `lym gate testmap` whenever "the last refresh reported stale packages left unbuilt".
   - Result: a new refresh starts every tick, holding a gate slot each time.
   - Contradiction: D6's bound says a dirty package "waits for the next prime HEAD change".
   - Stall: the record does not say whether skipped packages count against `--max`. If they do and the dirty packages come first in path order, each call builds nothing while still reporting packages left.
   - Evidence: `decision-record.md:85-95` (D5), `:103` and `:111` (D6).

3. **MEDIUM — design — D7's last paragraph, "The shipped template lists the step…"**
   - Summary: the template change never reaches a hub whose base `gate.yaml` already sets those keys, and this hub does.
   - Evidence: `_bolt/_lym/config/gate.yaml:46-51` sets `webster`, `parent-align`, `darn-write` and `code-burler-fix` whole, with no `reports`. `internal/gateslot/template.yaml:21-30` is where the step would be added.
   - Result: on this hub the step never runs, and the goal of stopping redundant tests from piling up is not met. No Decision says how a tuned hub gets `reports` onto its keys.

4. **LOW — design — D7 step 1**
   - Summary: the case with no merge base is unhandled.
   - Evidence: `internal/gatecli/scope.go:668-677`. `parentMergeBase` reports false for a worktree with no origin record or parent branch, such as the prime on `main`.
   - Gap: D7 says neither what step 1 judges then nor what the report says.

5. **LOW — design — D7 step 6**
   - Summary: for any package the run edits, "names … the changed files" also lists the diff's own files.
   - Why: those packages' files always differ from the map's hashes, so the run's own edits show up alongside drift on the parent.
   - Effect: the line cannot warn about map staleness, which is its purpose.
   - Evidence: `decision-record.md:129`.

6. **LOW — consistency — D7 step 6**
   - Summary: "contained with the fewest covering tests" claims a minimum that the reused algorithm does not compute.
   - Evidence: `cmd/testtiming/redundancy.go:186-235`. `coveringTests` picks the largest overlap first, which gives no minimum cover.
   - Fix: say "the covering tests, largest overlap first".

**Tool calls**

1. `grep` of `impactset.ChangedPaths` and `fabricengine.AnchoredFileAt` doc comments. My context had only their call and declaration lines, not what each returns, which D7 step 1 depends on.
