---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/run.go:532
    summary: No test pins that a failed engine-ledger write fails Start, removes the run directory and registers no strand.
    fix: In internal/shuttleengine/run_test.go, next to TestRunner_Start_AddStrandFailure_CleansRunDir, add a case that makes the ledger write fail (for example a directory at the run directory root's engines-run.json) and assert Start returns an error, AddStrand was never called, and runDirsUnder(root) is empty.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/config.go (migrateCoAuthoredBy, the value assignment and the claude entry node)
    summary: A co_authored_by that is empty, whitespace-only or not a scalar migrates to an engine_trailers.claude value of just the Co-Authored-By prefix, which TrailerBlock trims to a malformed bare trailer line.
    fix: In migrateCoAuthoredBy, when strings.TrimSpace(value) is empty, drop the co_authored_by pair without creating the claude entry (empty means off), report that in the rewrite line, and add a TestMigrateConfig row for it.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/run.go:532
    summary: If AddStrand fails after the ledger write, the run directory is removed but the ledger keeps the engine, so a landing can carry a trailer line for an engine that never started a session.
    fix: Either move the recordEngineRun call to after a successful AddStrand (before saveRunState), or state in recordEngineRun's doc comment that the ledger records start attempts that passed Prepare, and accept the over-count.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/enginedispatch_test.go (EnginesRun_counts_a_swept_run_and_a_record_without_an_engine)
    summary: Nothing runs the real orphan sweep to show it leaves engines-run.json and its lock file in place; the swept-run row removes the directory by hand.
    fix: Extend that subtest, or an existing sweep test, to run the sweep and assert engines-run.json and engines-run.json.lock survive and EnginesRun still returns the engine.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize.go (Step 1a2, the fz.deps.EnginesRun != nil guard)
    summary: The documented nil-means-no-engine-ran behaviour is never exercised, because every fixture sets EnginesRun.
    fix: Add a row to the Finalize message test with Deps.EnginesRun nil, wanting a landing message with no engine line.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/config_test.go (TestMigrateConfig)
    summary: No row covers the ErrInvalid paths (unparsable document, non-mapping root), and the existing-engine_trailers row uses an empty map, so keeping existing entries is not shown.
    fix: Add a row with a non-empty engine_trailers beside co_authored_by, asserting the entries are kept, and a check that an unparsable document returns an error marked configengine.ErrInvalid.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/landing/landing-template-describe.md:49 and contracts/stencils/landing/landing-template-describe-darn.md:46
    summary: Both stencils say the landing step appends its own trailer, but the trailer block is now operator-set and may be empty.
    fix: Reword both lines to say the landing step appends the configured trailer block itself, keeping the rule that the description carries no Co-Authored-By line.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/landingshed/deps.go (EnginesRun doc), internal/shuttleengine/enginesrun.go (EnginesRun and engineLedger docs)
    summary: New doc comments lean on other named symbols, such as CommitStatus's convention, loomcli's landingDeps and the ledger Start writes, against the self-contained comment rule.
    fix: Reword each comment to state only what the symbol does and why, dropping the references to CommitStatus, landingDeps and Start.
---

### [MEDIUM] Ledger write failure is untested

**Location:** internal/shuttleengine/run.go:532
**Issue:** The new failure branch in `start` removes the run directory and returns an error before `AddStrand`. The focus asked for exactly this behaviour, and no test reaches it.

### [LOW] Migration of an empty co_authored_by

**Location:** internal/landingshed/config.go, migrateCoAuthoredBy
**Issue:** An empty legacy value yields a bare `Co-Authored-By:` trailer after migration, where the board entry's "empty means off" would drop it. The old `LoadConfig` refused an empty value, but reconcile reads raw files.

### [LOW] Ledger over-records when AddStrand fails

**Location:** internal/shuttleengine/run.go:532
**Issue:** The board entry says an engine's text is included only when it ran at least one session. A start that fails at `AddStrand` leaves its engine in the ledger.

### [LOW] Orphan sweep leaving the ledger is unpinned

**Location:** internal/shuttleengine/enginedispatch_test.go
**Issue:** The claim that the sweep never removes the ledger rests on the sweep's directory-only loop, with no test of the real sweep.

### [LOW] Nil EnginesRun branch untested

**Location:** internal/landingshed/finalize.go, Step 1a2
**Issue:** The nil-seam branch has no fixture that reaches it.

### [LOW] MigrateConfig coverage gaps

**Location:** internal/landingshed/config_test.go
**Issue:** The error paths and the keep-existing-entries case are not pinned.

### [LOW] Describe stencils still describe a fixed trailer

**Location:** contracts/stencils/landing/landing-template-describe.md:49, contracts/stencils/landing/landing-template-describe-darn.md:46
**Issue:** "Appends its own trailer" no longer matches an operator-set, possibly empty block.

### [LOW] Doc comments depend on other symbols

**Location:** internal/landingshed/deps.go, internal/shuttleengine/enginesrun.go
**Issue:** Several new comments are written in terms of other named symbols.

## Rejected

- Integration test `"other"` engine entry restates a TrailerBlock row (code-tests NIT): a single integration-tier case, no named covering test that makes it redundant at that tier.
- PATTERN-leaf-packages unchecked (code-pattern-fit): the fork found no entry for the touched packages, so there is nothing to break.
