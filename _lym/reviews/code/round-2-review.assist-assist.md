---
verdict: APPROVED
findings:
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/driftengine/baseline.go:126
    summary: Stored partial-scope and old-check compatibility readers remain without a kept-item justification.
    fix: "Remove the obsolete stored partial-scope handling preserved by recordScope, its skip interpretation in internal/driftengine/check.go, and its rendering in internal/driftengine/verdict.go. Audit oldCheckLines and the old compare, waiver and acceptance fields with their readers and tests as the same compatibility-removal item. Preserve current full/off behavior and tolerant unknown-key handling. If live data requires any reader, establish that exception and name the reader and live-data reason under the kept items in _lym/landing/summary.md instead of treating stored records as outside the task."
    origin: lens:code-affected-set
    carries: F3
  - id: F8
    severity: MEDIUM
    class: consistency
    location: internal/driftengine/doc.go:469
    summary: The Drift-Baseline documentation still claims that partial config scope becomes full with a warning.
    fix: Remove the partial-to-full warning sentence from the Drift-Baseline section of internal/driftengine/doc.go and state that partial config scope is refused, matching checkScope and the removal of ScopeWasPartial.
    origin: lens:code-affected-set
    carries: F8
  - id: F10
    severity: LOW
    class: design
    location: internal/batcher/config.go:60
    summary: The default-layer retired-key remedy still offers fabric reconcile even though it preserves the offending key.
    fix: "For a template-named profile in LayerDefault, replace the lym fabric reconcile alternative with lym config reconcile --layer default --apply from the hub prime, or retain only hand deletion. Update internal/batcher/doc.go and the default-layer TestActive row to match. Keep the hand-delete-only remedy for operator-named profiles."
    origin: lens:code-affected-set
    carries: F10
  - id: F11
    severity: LOW
    class: design
    location: internal/configsync/configsync_test.go:429
    summary: Seed-only reconciliation lacks a case combining an absent hub file, a prime copy and an existing template base.
    fix: "Extend TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate with a template-base fixture field. Add a models row with absent hub models.yaml, a prime copy containing only an operator alias, and a current models.template-base. Assert that a missing template alias such as sonnet is added to the output and reported in Added, so removing !fileAbsent from configsync.go's FollowTemplate call fails the test."
    origin: lens:code-tests
    carries: F11
prior:
  - id: F1
    status: resolved
    answer: The listed contained tests now carry assertion-specific testtiming keep reasons.
  - id: F2
    status: resolved
    answer: ConfigSpec registers the retired plan keys with the tapestry way forward, which run-setting and profile validation reach before template validation.
  - id: F3
    status: open
    answer: The anchor fallback and old weft merge arms are removed, but the board scope covers old persisted data as well as config keys. The stored partial readers remain active and the kept list gives no live-data exception.
  - id: F4
    status: resolved
    answer: The remedy names the prime records branch; the test commits its marker and proves the retry adopts backend.
  - id: F5
    status: resolved
    answer: The Add integration test reaches the staged-only deletion path and asserts both the committed deletions and an empty index.
  - id: F6
    status: resolved
    answer: The integration table asserts exclude retention for a same-anchor link and removal for another anchor or a plain directory.
  - id: F7
    status: resolved
    answer: The test-only reinterpretation rollback test checks restored config bytes, absent marker and template base, and one subsequent marker commit.
  - id: F8
    status: open
    answer: The originally named overview, validator, fabric comment and stencil text are corrected; the changed drift package documentation still describes the deleted translation.
  - id: F9
    status: resolved
    answer: The summary calls the door one-way and explains the older-binary draft_status and retired-key consequences.
  - id: F10
    status: open
    answer: Operator-named profiles now get hand deletion alone. Template-named default-layer profiles still get a fabric reconcile alternative that leaves PruneUnknown false.
  - id: F11
    status: open
    answer: Registration and unbound-detail assertions are present. Existing exclude and parent-decision tests already cover those candidates. The prime-seed-with-existing-base combination remains absent.
---

### [MEDIUM] Stored drift compatibility remains

**Location:** internal/driftengine/baseline.go:126

**Issue:** An existing partial scope passes through `recordScope` unchanged.
`check.go:279` still applies its skipped-case semantics, and `verdict.go:102` renders the old scope.
`verdict.go:177` also interprets the old compare, waiver and acceptance fields.
Current scope creation produces full or off, and config loading refuses partial.
The board asks for removal of compatibility for old data, without limiting that instruction to config keys.
The summary's kept list gives no exception for these readers.

### [MEDIUM] Documentation describes a deleted translation

**Location:** internal/driftengine/doc.go:469

**Issue:** The Drift-Baseline section says partial config scope is recorded full with a warning.
`checkScope` now refuses partial, and the baseline no longer translates or warns.

### [LOW] Default-layer recovery leaves the refusal in place

**Location:** internal/batcher/config.go:60

**Issue:** A template profile carrying `alone_above` is offered `lym fabric reconcile` as a removal route.
That verb passes `HubWideOptions{Apply: true}` at `internal/fabriccli/fabric.go:670`.
With `PruneUnknown` false, `pruneSparse` preserves the key, so the load refuses again.
The explicit default-layer config reconcile sets `PruneUnknown` true at `internal/configcli/hubwide.go:193`.

### [LOW] Prime seed does not test an existing template base

**Location:** internal/configsync/configsync_test.go:429

**Issue:** The prime-seed row creates hub and prime config files but no template base.
It therefore cannot distinguish `templateBasePresent && !fileAbsent` from `templateBasePresent`.
Treating the prime copy as a present hub file can preserve a missing template alias as an operator deletion.

## Rejected

- The F11 exclude-content candidate already has meaningful-entry membership/count checks and byte-idempotence checks.
- The F11 manual-decision candidate is covered by `TestBouncer_AutoAccept`'s parent-accept row and automatic-flag check.
- The correctness lens's F10 resolution overlooks the default-layer call's false `PruneUnknown`; F10 remains open.
- The three lenses' stored-drift findings describe one remaining enumeration gap and are consolidated as F3.

The verdict uses the required severity mapping: no BLOCKING-severity finding was established.
The carried MEDIUM and LOW findings remain open.
`go build ./internal/driftengine ./internal/configreg` passed; this review did not rerun the test suite.
