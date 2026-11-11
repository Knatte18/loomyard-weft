---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/websterengine/verifygatereport.go:217
    summary: The findings text Merriam reads still says attempt 4 of 3, the off-by-one the stuck reason no longer has
    fix: >-
      In renderVerifyGateFindings, replace the line "Verify gate failed (attempt %d of %d)." so the count and the limit agree.
      Print the report's Attempt as the evaluation number and Cap as the re-prompt limit,
      for example "Verify gate failed (evaluation %d; the session may be re-prompted %d time(s))."
      Update the matching "attempt 1 of 3" assertions in verifygatereport_test.go and in the gate tests that read the rendered findings.
  - id: F2
    severity: LOW
    class: consistency
    location: contracts/stencils/webster/webster-template-master.md:193
    summary: The probe-failure refusal tells Master to re-run the verb, but the Master stencil has no rung for it
    fix: >-
      Under the `run_ended` rung in webster-template-master.md, add a rung for a bracket verb that refuses
      with the message "could not tell whether a live `lym webster run` owns this state" and no flag.
      It tells Master that the refusal is transient and that it re-runs the verb once;
      if the second call refuses the same way, Master writes `outcome: stuck` with a `stuck_reason` quoting the refusal's message and stops.
---

### [MEDIUM] Findings text keeps the attempt-over-limit wording

**Location:** internal/websterengine/verifygatereport.go:217
**Issue:** Issue #106 asks that the attempt count and the limit agree.
The diff redefines `Cap` as the entry's re-prompt budget, so `Attempt` can be `Cap + 1`, and the `VerifyGateReport.Cap` comment says so.
The stuck reason now counts re-prompts against the limit.
`renderVerifyGateFindings` still prints `attempt %d of %d` from the same two fields, so the text Merriam and the fixer read can say "attempt 4 of 3".
The round-2 focus on this function is met for the no-test-named case: with `Failures` empty and `LogTail` set it prints the exit code, the log path and the tail, with no identity list and no card hint.
This finding is about the header line above that branch.

### [LOW] No Master rung for the probe-failure refusal

**Location:** contracts/stencils/webster/webster-template-master.md:193
**Issue:** `orphanedMasterRefusal` answers a failed `RunActive` probe with a refusal that has no `run_ended` and the way forward "re-run the verb".
The code meets the focus question: a probe error never carries `run_ended: true`, which `TestRefuseOrphanedMaster_RefusesAMasterWhoseRunEnded` pins.
The stencil names the `run_ended` rung and the `config_invalid` rung with its "could not be read ... re-run the verb" line, and nothing for this refusal.
The stencil also says Master never retries a refused verb beyond the steps it lists.
A Master reading both has no listed step for this message.

## Rejected

- The six bracket verbs are tested through the real verb tree by `addVerbs`, with a free `run.lock` and a Master strand name, and `status` is shown not to be refused.
  The test is not parallel and says why in its doc comment. No finding.
- The stuck-reason branches have rows: `failedEntryReprompts` with and without a failed entry, `cutHead` and `cutTail` on a rune boundary and at the limit, the per-arrival `command` reset, and an empty output with no dangling tail clause. No finding.
- `docs/overview.md` and the Long help of `lym loom decision add` both list `notice_held_back` beside `notified` and `notice_errors`. No finding.
- The Long help example of `decision add` was shortened to fit the byte cap. It still shows the required flags. No finding.
