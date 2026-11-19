---
verdict: BLOCKING
findings:
  - id: "F1"
    severity: "BLOCKING"
    class: "design"
    location: "internal/shedadapters/bouncer.go:2067"
    summary: "Recovered Bouncer verdicts bypass the parent-findings refusal, so a crash can let CONVERGED approve a round over the parent's findings."
    fix: "Move the parent-findings refusal from judgeCall into the shared settlement path in bouncer.go. Resolve the round's held ask from its generation seed without sending another ask, and enforce the refusal on fresh harvest, live-judge recovery and entryHarvest before approving the segment. Extend the Bouncer recovery tests with a done parent ask holding findings and an otherwise converged verdict."
    origin: "lens:code-correctness"
    carries: "F1"
  - id: "F15"
    severity: "BLOCKING"
    class: "design"
    location: "internal/shedadapters/parentseat.go:62"
    summary: "The new judgeParentBlock embeds Bouncer instructions in Go and bypasses PATTERN-stencil-ownership's required call-time stencil registry."
    fix: "Move judgeParentBlock into a registered Bouncer stencil fragment under contracts/stencils/bouncer. Read and render it through stencilstore at judge-call time when the round holds a parent ask, and compose that rendered fragment into the judge prompt in bouncer.go instead of appending a Go constant."
    origin: "lens:code-affected-set"
  - id: "F5"
    severity: "MEDIUM"
    class: "design"
    location: "internal/driftengine/verify.go:846"
    summary: "Command-mode drift never initializes judgeStartedAt, so parentAgreement supplies no deadline cap and a silent parent can still time out the judge."
    fix: "Initialize judgeStartedAt once on the shared judge-run path before tableWithParent is built, covering both judge and passCommand on fresh runs and resumes, and remove the line-mode-only initialization. Extend the production-path test to evaluate the parent entry for command mode after the clock advances and assert that its deadline is before the judge timeout."
    origin: "lens:code-correctness"
    carries: "F5"
prior:
  - id: F1
    status: open
    answer: "The ask now names the current round's files, but recovered verdicts still bypass the parent-findings check."
  - id: F2
    status: resolved
    answer: "The ask records a digest of parsed rulings; changed rulings after satisfied or findings answers send a new ask."
  - id: F3
    status: resolved
    answer: "Resolved by \"Opt-in parent-ask keys stay outside Guard\"; the declared exception is present in PATTERN-config-layers."
  - id: F4
    status: resolved
    answer: "Resolved by \"Drift key names stand\"; the approved drift.yaml names remain unchanged."
  - id: F5
    status: open
    answer: "The run-start cap fixes line mode, but passCommand never initializes the timestamp consumed by parentAgreement."
  - id: F6
    status: resolved
    answer: "TestParentSeats_BuildEachSeatFromItsOwnPerch asserts both seats' fields, setting keys and waits."
  - id: F7
    status: resolved
    answer: "TestDriftVerify_ParentAgreementCarriedWhileAsking asserts the ordered judgement and parent-agreement entries."
  - id: F8
    status: resolved
    answer: "The three requested keep markers are present."
  - id: F9
    status: resolved
    answer: "TestParentAgreementGate now covers pending-post recovery, resolution failure and the answer-expiry race."
  - id: F10
    status: resolved
    answer: "The replacement direct-ask test covers rounds 1 and 2 naming their own reports, consistent with the moved checkpoint."
  - id: F11
    status: resolved
    answer: "TestDriftDeps_BuildTheJudgesParentSeat asserts the anchor-based files, slug and routing seams."
  - id: F12
    status: resolved
    answer: "The not-latest refusal tells the caller to review what the newer ask names."
  - id: F13
    status: resolved
    answer: "ParentAsk and Seed document both review-generation and drift-spawn asks."
  - id: F14
    status: resolved
    answer: "Discussion selection uses its own tag and recognizes legacy untagged asks; the foreign-ask rows cover this."
---

### [BLOCKING] Recovered verdicts can discard parent findings

**Location:** internal/shedadapters/bouncer.go:2067
**Issue:** The new refusal runs only in the fresh judge-call harvest.
If the parent answers with findings and the driver dies after the judge writes CONVERGED,
the next call can recover that verdict through the live-judge harvest or entryHarvest.
Those paths call settleUnlessRefused directly at bouncer.go:453 and bouncer.go:477.
Its checks at bouncer.go:1062 do not consult the parent's report, so the segment can be approved over those findings.
This leaves F1 open despite the corrected ask timing and report paths.

### [BLOCKING] Bouncer instructions bypass stencil ownership

**Location:** internal/shedadapters/parentseat.go:62
**Issue:** judgeParentBlock is an embedded instruction section appended at bouncer.go:2010.
PATTERN-stencil-ownership requires prompt text to be read at call time through stencilstore.
Its orch-role exception does not cover Bouncer.
The registry-backed judge stencil cannot own these instructions, and no departure trailer permits this change.
The constraint explicitly assigns this check to review.

### [MEDIUM] Command-mode drift still has no parent deadline cap

**Location:** internal/driftengine/verify.go:846
**Issue:** judgeStartedAt is assigned only by judge at verify.go:456.
The command-mode path instead calls passCommand and reaches runAgent at verify.go:1071 without assigning it.
judgeAskLimit treats that zero value as no limit.
A judge that starts at 12:00 and presents rulings at 12:40 therefore gets a 13:10 parent deadline by default,
although JudgeTable's session timeout ends at 13:00.
The silent-parent failure described by F5 remains reachable in command mode.

## Rejected

- The duplicate command-mode deadline report from code-affected-set is consolidated into F5.
- Missing production-path deadline assertions support F5; they are not a separate new BLOCKING finding.
- The contained-test keep-marker candidates are below this incremental round's threshold for new findings.
  Containment alone does not justify deleting tests that assert distinct paths, identities or round-2 report names.
- The stale landing summary is not a newly changed hunk in this incremental round.

## Review checks

The digest check preserves an answer only for the parsed rulings it presented.
A changed judgement after a satisfied or findings answer sends another ask.
Discussion ask selection uses its own latest tagged ask, with a fallback for legacy untagged asks.
The focused test additions use injected clocks and do not add production-duration sleeps.

The named packages compiled with:
`go build ./internal/driftengine ./internal/shedadapters ./internal/loomcli`.
No test suite was run in this review.
The available shell tool exposes no command-timeout parameter for the required gate invocation.

