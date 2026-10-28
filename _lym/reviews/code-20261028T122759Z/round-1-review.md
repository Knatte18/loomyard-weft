---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/fabricengine/refscanner_test.go:68
    summary: No scanner row pins that gate blanking applies only in command position and stops at each of semicolon, double pipe, pipe and newline
    fix: In TestRefScanner_Matches add rows that must match (want true) for "echo lym gate other-weft", "lym gate test --card a-weft; ls other-weft", "lym gate test --card a-weft | grep other-weft", "lym gate test --card a-weft || ls other-weft" and a newline-separated "lym gate test --card a-weft\nls other-weft".
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/websterengine/recoverbatch.go:574
    summary: acceptAuditWayForward has no row for the dirty-tree refusal, the HEAD-not-descended refusal or a git read failure
    fix: In TestRecoverSpawnOrAttach add a row for a mutating fabric reference with a recorded start and a dirty worktree, asserting the way forward names accept-audit's dirty-tree reason, the re-run of accept-audit then recover-batch and the appended reset-to-start route, and a row for a HEAD that does not descend from the start; add a row whose git fake returns a read error, asserting the error is returned and is not ErrRecoveryNeedsFresh.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/fabricengine/refscanner.go:29
    summary: The gate segment ends at the ampersand of a 2>&1 redirect, so a weft-suffixed card ID written after the redirect is still flagged
    fix: In gateSegmentPattern keep a redirect ampersand inside the segment, for example by making the trailing class a repetition of "(?:[0-9]*[<>]&[0-9-]*|&>|[^;&|()\n`])" in place of "[^;&|()\n`]", and add a row "lym gate test 2>&1 --card 6-prime-weft" with want false; or document the limit in ReferenceRule's doc comment.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: consistency
    location: internal/planparser/fabricref.go:74
    summary: The card-ID finding still says every command naming the card fails the audit, and still rejects the ID, though a lym gate command naming it no longer fails; the change description's claim that the plan check no longer flags it is only true of Verify spans
    fix: Narrow the card-ID finding text at fabricref.go:74 to say the ID is read as a weft sibling by any command other than a lym gate invocation, and correct the sentence in _lym/landing/summary.md about the plan check to say only a Verify span with a gate command is no longer flagged; a planparser test row for a Verify gate command naming a suffix-ending card ID may pin it.
    origin: lens:code-affected-set
  - id: F5
    severity: NIT
    class: design
    location: internal/websterengine/auditledger.go:263
    summary: The readOnly nil guard in batchFabricReferenceEvidence has no test row
    fix: Add a recover-batch row that leaves Deps.ReadOnly nil on a committed batch and expects the batch-start route, or drop the nil check.
    origin: lens:code-tests
---

### [LOW] Gate blanking bounds are unpinned

**Location:** internal/fabricengine/refscanner_test.go:68
**Issue:** The four new rows pin the incident command, the chained-after-commit form, a separator-prefixed path and a bare name after `&&`. Nothing fails if the segment ran past `|`, `;`, `||` or a newline, or if blanking applied outside command position, which is the D1 bound the acceptance criteria name.

### [LOW] acceptAuditWayForward refusal branches untested

**Location:** internal/websterengine/recoverbatch.go:574
**Issue:** The no-start, at-start and past-start branches have rows. The dirty-tree and non-descendant refusals, and the pass-through of a git read failure, are reached by no recover-batch test, so the `stepResetToStart` append logic is exercised only for the branches that already name it.

### [LOW] Redirect ampersand ends the gate segment

**Location:** internal/fabricengine/refscanner.go:29
**Issue:** `[^;&|()\n`]*` treats the `&` in `2>&1` as a separator, so arguments after a redirect fall outside the blanked segment. The result is a false positive only; the incident command puts the redirect last and is handled.

### [LOW] Plan-check message and summary claim drift from the gate exemption

**Location:** internal/planparser/fabricref.go:74
**Issue:** The ID itself is still rejected, with text claiming every command naming it fails the audit, which is no longer true for `lym gate`. The summary's claim that the plan check no longer flags a Verify naming the ID holds only for Verify and code spans holding a gate command.

### [NIT] Untested nil guard

**Location:** internal/websterengine/auditledger.go:263
**Issue:** `readOnly == nil` is a new behavior of the extracted function with no row.

## Rejected

- Lens code-correctness and code-affected-set found no defect in the `batchFabricReferenceEvidence` split, `PlanReset` text, the docs hunks or PATTERN fit; I checked the `recoverbatch.go` call site and the grep for the old reset text, and the only remaining "wait for it to finish" is `runlevel.go:408` for the `run` verb, out of scope.
- Missing `report-head` or other-guard coverage: the reset test keeps its `report-head` pass-through assertion, so no finding.
