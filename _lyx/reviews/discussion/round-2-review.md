---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:54
    summary: D4 says the refusal happens before any run directory exists, but the shuttle runner creates the run directory before Prepare and removes it on a Prepare error
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:52
    summary: D4 refuses an empty default TTL but never says whether a zero-value Config literal counts, which every existing Prepare fixture passes
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:104
    summary: The seam acceptance criterion says the existing seam enforcement test stays green, but that test checks imports only and cannot enforce the vocabulary rule
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:73
    summary: The Constraints entry and D5 cite PATTERN-spawn-observability as the source of the TTL log line, but Prepare starts no process and the pattern does not require it
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md:20
    summary: The rejected nested-key alternative is dismissed for unclear merge semantics, which applies equally to the chosen flat map and is resolved there by the open-map declaration
---

### [LOW] D4 misplaces the refusal relative to the run directory

**Location:** `_lyx/discussion/decision-record.md:54` (D4 Bound).
**Issue:** D4 states the refusal "happens in `Prepare`, before any pane or run directory exists".
In `internal/shuttleengine/run.go` (`start`), `createRunDir` runs before `r.engine.Prepare`, and a `Prepare` error is followed by `os.RemoveAll(runDir)`.
The run directory therefore exists when the refusal fires and is removed by the runner.
What holds is that `Prepare` validates before writing any artifact into it, and before `AddStrand` creates a pane.
The same bound says the guard never touches a "started or rebuilt session"; a run that adopts an existing session by `Spec.ResumeSessionID` also goes through `Prepare`, so it is a new run and is refused.
That is the right behavior, but the bound does not say so.
**Fix:** reword the Bound to "the refusal happens in `Prepare`, before any artifact is written and before reed creates a pane", and say it blocks a new run, including one that adopts a session by id.

### [LOW] D4 leaves the zero-value Config open

**Location:** `_lyx/discussion/decision-record.md:52` (D4 Decision) and `:102` (Acceptance criteria).
**Issue:** D4 and the acceptance criteria refuse an empty `claude_prompt_cache_ttl`.
A `shuttleengine.Config{}` literal has the empty string in both new fields, and the existing `Prepare` tests build exactly that (`New().Prepare(runDir, spec, shuttleengine.Config{})`), as do fixtures outside `claudeengine`.
Read literally, all of them are refused after this change.
The record does not choose between treating the zero value as the refused empty value and tolerating it.
That choice decides whether the fixtures build their `Config` from the template defaults or `Prepare` gains a zero-value branch.
**Fix:** add one sentence to D4 stating that an empty value is refused whatever its source, a programmatic `Config{}` included, and that test fixtures set the TTL fields from the template defaults.

### [LOW] The seam acceptance criterion names a test that cannot enforce it

**Location:** `_lyx/discussion/decision-record.md:104` (Acceptance criteria).
**Issue:** the criterion reads "No `shuttleengine` (non-`claudeengine`) file names `CLAUDE_CODE_PROMPT_CACHE_TTL` or the `5m|1h` vocabulary beyond the template defaults; the existing seam enforcement test stays green".
`TestProviderSeamImportRule` in `internal/shuttleengine/seam_enforcement_test.go` only checks that no file imports `claudeengine`.
It stays green whatever strings `shuttleengine` holds, so the criterion gives the plan writer no observable for the vocabulary half.
`PATTERN-shuttle-provider-seam` is enforced for vocabulary by review.
**Fix:** state the vocabulary half as a review-enforced constraint, already carried by the Constraints entry for the seam, and keep only the import test as the mechanical check.

### [NIT] PATTERN-spawn-observability is cited for a log line it does not require

**Location:** `_lyx/discussion/decision-record.md:60` (D5) and `:73` (Constraints).
**Issue:** `PATTERN-spawn-observability` covers code that starts a real OS process and logs its spawn and teardown.
`claudeengine.Prepare` starts no process; reed spawns the pane.
The log line is a sound choice on its own grounds (the resolved TTL exists only inside `Prepare`), but citing the pattern presents it as a pattern-mandated constraint.
**Fix:** drop the pattern citation from D5 and from the Constraints entry, or phrase it as consistent with the pattern's intent, and keep the Constraints entry as the log requirement it is.

### [NIT] The nested-key rejection rests on a hazard the chosen design also has

**Location:** `_lyx/discussion/support-log.md:20` (Rejected alternatives).
**Issue:** the entry rejects `claude_prompt_cache_ttl: {default, roles}` because partial nested maps have "unclear merge semantics" under the degrading loader, and says two flat keys keep the missing-key rule exact.
The chosen `claude_prompt_cache_ttl_roles` map has the same merge hazard, and Q2 in the ledger resolves it by declaring the key an open map, which a dotted `claude_prompt_cache_ttl.roles` path would equally allow.
The stated reason does not separate the two designs.
**Fix:** restate the reason as what actually differs, that the flat keys follow the existing `claude_*` split and keep the default a plain scalar, and drop the merge-semantics claim.

## Focus questions settled

- Merge semantics: `yamlengine.FillMissing` descends into present mappings and appends template keys the file lacks, skipping a key at a declared open-map path that the file holds.
  `LoadConfig` currently passes no open maps to `LoadOrTemplate`, and `batcher.ConfigOpenMaps` plus its `configreg` entry are the precedent D2 follows.
  D2, Scope and the acceptance criteria agree with that behavior; no finding.
- Whole-set refusal: D4 states what the guard blocks (a new run of any role), what it cannot touch (a started or rebuilt session, whose resume line is stored) and what bounds it (one edit to `shuttle.yaml`, an error naming the file).
  The only defect is the run-directory claim in F1.
- Log placement: D5 names `claudeengine.Prepare` as the logger, and the acceptance criterion gives the test an observable, the captured logger output.
  `claudeengine` already imports `internal/logger` and its tests already capture output with `logger.SetOutput`; no finding.
- Resolution timing: the Goal, D1 and the acceptance criteria agree that the TTL is resolved once in `Prepare` and both lines carry it; reed replays the stored `ResumeCmd`, and no other code builds a resume line.
  No finding.
