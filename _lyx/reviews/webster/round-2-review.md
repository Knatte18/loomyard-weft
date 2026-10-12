---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/loomcli/smoke_driverstrand_test.go:213
    summary: TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps still wants 0 status strands after an llm-seeded start, but card 7 makes start ensure the loom-status strand on both drivers
    fix: In internal/loomcli/smoke_driverstrand_test.go change the three statusStrandCount checks (after the first bootstrap near line 213, the second near line 229, the third near line 263) from count != 0 to count != 1, and reword their messages to "want exactly 1 -- an llm-seeded start ensures the status strand". Reword the file comment near line 11 and the test doc comment at lines 170-171 to say an llm-seeded start leaves exactly one status strand beside its driver strand.
  - id: F2
    severity: BLOCKING
    class: scope
    location: internal/loomcli/smoke_starttail_test.go:84
    summary: TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand still wants 0 status strands after an llm-driven start, but card 7 makes start ensure the loom-status strand on both drivers
    fix: In internal/loomcli/smoke_starttail_test.go change the check at line 84 to count != 1 with the message "status strands after an llm-driven start = %d; want 1". Update the file comment at line 4 (neither a loom-operator strand nor a status strand) and the test doc comment at lines 39-41 (no status strand, an llm-driven start carries no status strand) to say the start leaves exactly one status strand and still no loom-operator strand.
---

### [BLOCKING] TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps fails on a stale status-strand assertion

**Location:** internal/loomcli/smoke_driverstrand_test.go:213
**Issue:** The Publish verify log (`.lyx/verify/verify-6.log`, line 89) reports `status strands after the first bootstrap = 1; want 0`.
The merge-in commit `900008660` did not cause it: the branch's own card 7 (`7615d031f`) changed `startCmd` to call `ensureStatusStrand` on both drivers and removed `applyStatusStrandSurface` and `removeStatusStrands`.
Card 7 edited only `TestSmokeStatusStrandAcrossDriverSeeds` in `smoke_drivingsurface_test.go`, which now wants exactly one status strand after an llm-seeded start.
This test and its sibling in `smoke_starttail_test.go` carry the same old llm-arm assertion and were left unedited.
The plan's Edit list for card 7 did not name them, so the gap is in the diff's coverage of the behavior change, not in the production code, which matches the card.

### [BLOCKING] TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand fails on a stale status-strand assertion

**Location:** internal/loomcli/smoke_starttail_test.go:84
**Issue:** The Publish verify log (line 93) reports `status strands after an llm-driven start = 1; want 0`.
It has the same cause as F1: card 7 makes the llm-seeded start keep the `loom-status` strand, and this assertion and the comments around it still describe the removed llm arm.
The test's other property, that no `loom-operator` strand exists and no tmux client is attached, is unaffected and stays.

## Rejected

- The merge-in diff (`900008660^1..900008660`) is not the cause of either failure; both tests fail on assertions about card 7's behavior inside `internal/loomcli`, which that merge did not touch.
- The tmux and llm tiers cannot be run in this session, so the diagnosis rests on the log and on reading the card, the production code and the assertions; no other part of the branch's range was re-derived beyond that.
