---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/stencilstore/skew.go:71
    summary: Runtime fallback renders embedded prompts despite PATTERN-stencil-ownership forbidding it.
    fix: Replace the embedded-content return with an early skew error propagated by Read and readInclude that names the stencil and directs restarting the driver on the current binary; remove unused fallback registration and update the changed documentation to describe refusal.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/stencilstore/skew_test.go:53
    summary: The skew regression tests a private helper instead of the public read behavior required by PATTERN-test-speed.
    fix: Move the skew cases into public Read, ReadSet and ReadForEngine behavior coverage using controlled registry and running-identity inputs; cover a newer main stencil, plain include and engine block, assert the final skew diagnostic and unchanged disk content, and cover repeated concurrent diagnostics if suppression remains.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/stencilstore/skew.go:68
    summary: A known newer edited copy bypasses both safe fallback and the required actionable skew error.
    fix: Keep the operator file unchanged but return a skew error for a differing newer copy when its edit prevents safe substitution; propagate that error through both stencil and include reads with the restart remedy, and change the edited-newer regression expectation accordingly.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/stencilstore/skew.go:65
    summary: Timestamp ordering misses descendant builds that production refresh permits with equal or earlier commit times.
    fix: Make skew detection agree with refresh eligibility using a caller-supplied version relation or a conservative incompatible-revision refusal; do not run git inside stencilstore, and cover refreshed descendant copies with equal and earlier commit times through the public read path.
---

### [BLOCKING] Embedded rendering conflicts with the repository constraint

**Location:** internal/stencilstore/skew.go:71

**Issue:** `PATTERN-stencil-ownership` explicitly prohibits runtime prompts from embedded bytes.
The registry registered at `contracts/stencils/stencils.go:1142` supplies those bytes to the new return path.
The exploration instructions say repository constraints prevail over conflicting scope.
The Sketch permits an early error with a restart remedy, so the task has a compliant implementation available.
No declared PATTERN change or departure trailer changes that rule in this range.

The affected-set lens also found dependent claims that become false under fallback:
`internal/shedadapters/bouncer.go:330` says reads never fall back,
and `internal/parentdirective/doc.go:34` says its text is never embedded.
`internal/loomcli/stencilpins.go:65` promises changed prompts when disk copies change.
These are consequences of the same source-policy change and are consolidated here.
Removing fallback restores the disk-source guarantees; any retained source-policy change must reconcile those claims.

### [BLOCKING] The regression bypasses the changed public paths

**Location:** internal/stencilstore/skew_test.go:53

**Issue:** `TestShippedOverNewer` supplies its own registry and writer directly to a private helper.
It stays green if registration or either runtime call to `contentForRunning` is removed.
Existing `TestReadSet` and `TestReadForEngine` fixtures have no skew metadata.
They therefore do not assert the new public behavior despite reaching the reader functions.
`PATTERN-test-speed` requires exported-surface behavior tests and expressly rejects private-helper substitutes.
The supplied reach report does not establish that the fallback branch is exercised through those paths.

The code-tests lens's separate diagnostic finding is consolidated here:
no assertion checks the new warning fields, restart guidance, or once-per-stencil behavior under repeated reads.
Coverage must follow the final skew policy, including error propagation if refusal replaces fallback.
The existing skew table has no waits or production-duration dependence and is not redundant with another named test.

### [MEDIUM] Edited newer text still reaches the incompatible renderer

**Location:** internal/stencilstore/skew.go:68

**Issue:** Consider a newer stamped body introducing `{{.new_marker}}`, followed by an operator's appended sentence.
Its recorded writer is later, but the body no longer matches its stamp.
`Classify` returns `StateEdited`, and the new branch returns the disk bytes without a warning or error.
`gatefixer.RenderPrompt` then reaches `stencil.FillOptional` with no value for the new marker.
The result remains an unfilled-marker failure rather than the Sketch's skew diagnostic and restart remedy.
An edited include can likewise be combined with a shipped parent body.
Preserving the edit requires leaving the file alone, not passing incompatible content onward.
The code-scope lens reported the same omission; it is retained once as a design finding.

### [MEDIUM] The reader's ordering differs from the writer's ordering

**Location:** internal/stencilstore/skew.go:65

**Issue:** `fabricengine.StencilSource` uses strict ancestry when both revisions exist.
Its ordering permits a descendant binary to refresh the ancestor's board copy even with equal commit times.
The integration test at `internal/fabricengine/warpforward_integration_test.go:211` also pins ancestry over reversed times.
The still-running ancestor's new reader uses only `Time.After` and therefore accepts that refreshed body unchanged.
If it introduces a required marker, the old renderer still fails without identifying skew.
The refresh writer and runtime reader need compatible version semantics.

## Focus results

`Classify` returns `StateUntouched` only when a parseable stamp matches the normalized body hash.
A malformed stamp cannot trigger fallback for a differing body; absent writer time also prevents fallback.
`Read`, plain includes, and engine includes all reach the new selection function.
The remaining direct disk reads inspect or reconcile board state rather than render it.
Engine block names match the registry's `engine/<engine>/<block>` keys.

Registration has one package-init caller; repeated imports do not repeat initialization.
There is no concurrent registration in the actual call graph.
`sync.OnceValue` protects the running identity, and `sync.Map.LoadOrStore` suppresses concurrent duplicate warnings.
An unstamped running binary takes the unchanged path because its time is zero.
A later-time unrelated writer selects the running binary's own text under the implemented policy;
the evidence does not establish a separate concurrency or branch-origin defect.
The changed documentation describes that policy, subject to F1 and its detection limits.

## Validation

Reviewed the supplied diff pieces and commit messages for `c5a0ea745f362a36974fda25ba0c3f917f152684..HEAD`.
Read the board entry and amended Sketch; no obsolete include-wiring work or duplicate gatefix test is required.
All lens reports are saved beside this report.
The focused tier0 command could not acquire a gate slot:
the gate holder-record directory under the hub's `_bolt/.lym/gate` is read-only in this session.
No test result is claimed from that attempt; the supplied gate reports remain the available execution evidence.
