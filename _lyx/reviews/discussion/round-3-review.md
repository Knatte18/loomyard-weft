---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:136
    summary: "D8's attach way forward says the next probe answers not found after `lyx reed remove`, but the probe errors for a record younger than twice startup_timeout_s"
    fix: "In D8's attach-refusal bullet (line 136), replace 'so the next probe answers not found and the caller's `Start` uses the spec's engine' with a statement that matches attach.go. `lyx reed remove` leaves the run record reading running, so the strand becomes untracked and the probe applies its age rule. The probe answers not found only once the record is older than twice `startup_timeout_s`, and before that it errors with the attach.go 'younger than' message that tells the caller to wait. Make the way forward 'end the run's live strand with `lyx reed remove <strand guid>`, then re-step the row once the record is older than twice `startup_timeout_s`', and extend the acceptance bullet for the attach refusal (the one at line 234) to assert that the refusal names that wait."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:138
    summary: "D8 says the attach refusal adds no loom or webster refusal row, but an attach refusal with the same way forward already has a loom row, and the record gives no reachability reason for skipping one"
    fix: "In D8 (line 138) keep the 'shuttle error' statement but replace 'so it adds no loom or webster refusal row' with the reason it holds. The refusal is unreachable from any on-disk state while Claude is the one registered engine and declares all three requirements and no role sets `Requires` beyond the `ForkSubagents` implication. Add an R-item to Open risks naming `engine-model-choice` and `codex-engine` as the tasks that land the loom-section refusal-spec row (the way forward is the F1 text) with the first reachable `Requires` or second engine. Alternatively, add the row to `contracts/specs/refusal-spec.md` in this commit beside the existing 'producer re-run: superseded strand not removed' row, and list `contracts/specs/refusal-spec.md` in the Docs-land-in-the-same-commit constraint."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:81
    summary: D4's attach bullet says an unmet requirement refuses the attach without the finished-run exemption that only D8 states
    fix: "In D4's attach bullet, after 'When the recorded engine does not meet them, the attach is refused with D8's error and the attach way forward D8 names', add 'except a finished run, which D8 harvests without the check'. Do not restate the exemption's reason; D8 owns it."
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:122
    summary: D7's Bound and R4 each state a different part of when Webster's audit engine can differ from the engine that ran a batch, so the boundary is stated twice
    fix: "Make D7's Bound the single statement of the conditions: the audit and parse engine equals the engine that ran a batch's sessions only while one engine is registered; with two registered it can differ when a Webster spawn spec names an engine or when `shuttle.yaml`'s `engine:` changes between a batch's spawn and its later audit, recover parse or archive. Cut R4 to the owners: `engine-model-choice` keeps Merriam and batch specs on the default or resolves the engine from the batch run's record, and `codex-engine` resolves Webster's audit and parse engine per batch."
---

### [MEDIUM] D8's attach way forward does not clear the refusal at once

**Location:** `_lyx/discussion/decision-record.md:136`

**Issue:** D8 gives the attach refusal this way forward: end the live strand with `lyx reed remove <strand guid>`, then re-step the row, "so the next probe answers not found".
I read `internal/shuttleengine/attach.go` to check that claim.
`lyx reed remove` removes the strand from reed and leaves `run.json` reading running, since only shuttle's stop verb settles a record (`pattern/PATTERN-shuttle-stop.md`), and no `lyx shuttle` verb exposes it.
After the removal the candidate is untracked, so `dispositionCandidate` falls to `leftoverThenAgeVerdict`.
With the output files absent and the outcome running, a record younger than `minAge` (twice `startup_timeout_s`, so 180 s at the template's 90) is `verdictError`, and `attach` returns the "cannot be confirmed dead or alive" error rather than not found.
Only an older record is respawn-eligible and answers not found.
The step is named, is not in the denied list (`pattern/PATTERN-no-denied-recovery.md`), and does clear the refusal, but only after that wait.
The decision states an unconditional effect that the code does not give, and the acceptance criteria would not catch it.
The attach error carries its own "wait" way forward, so the caller is not stranded.
That is why this is MEDIUM and not BLOCKING.

### [MEDIUM] D8 skips a refusal-spec row without saying why that holds

**Location:** `_lyx/discussion/decision-record.md:138`

**Issue:** D8 says a loom or webster driver that hits the attach refusal "surfaces it through its existing spawn-failure path, so it adds no loom or webster refusal row".
`pattern/PATTERN-refusal-way-forward.md` requires a row in the module's section for every refusal reachable through a bound module's verbs.
`contracts/specs/refusal-spec.md` already carries the loom row "producer re-run: superseded strand not removed" for `attach.go`'s own removal failure, a shuttle error with the same `lyx reed remove <guid>`, then re-step way forward.
So the table does list shuttle-raised refusals that producer rows reach, and I found no generic spawn-failure row in it that D8's claim could rest on.
The skip can still be right: with Claude the only registered engine, declaring all three requirements, and no role's `Requires` set (D8's last Decision bullet), neither the Start nor the attach refusal is reachable from any state a run can produce.
The record does not say so, and it names no task that lands the row once the refusal becomes reachable.
The Start refusal's alias way forward has the same gap, and the same fix covers it.

### [LOW] D4 omits the finished-run exemption from its attach refusal

**Location:** `_lyx/discussion/decision-record.md:81`

**Issue:** D4 says that when the recorded engine does not meet the joined requirements "the attach is refused", with no exception.
D8 and the acceptance criteria say a finished run, every declared output file on disk, is harvested without the check.
That matches `dispositionCandidate`, which makes a running record with every output file on disk attachable before any reed read.
A plan writer who works from D4 alone, as the owner of the attach path, would miss the exemption.
D8 owns the reason, so D4 only needs the exception clause.

### [NIT] Webster's audit-engine divergence is stated in two places

**Location:** `_lyx/discussion/decision-record.md:122`

**Issue:** D7's Bound covers the registered-engine count and a Webster spawn spec naming an engine.
R4 repeats the spec-names-an-engine condition and adds the `engine:` change across a Merriam restart or recover.
The support log says the changed-default condition was fixed "in D7's bound and R4", but D7's Bound does not carry it.
Every condition and its owner is stated somewhere (R4 names `engine-model-choice` and `codex-engine`), so the focus question is settled and nothing is missing.
The split only makes the boundary harder to maintain.

## Focus directive verdicts

None of these departs from the directive.

- **Attach path and fork audit.**
  D8's bound holds for a record carrying `ForkSubagents` attached by a spec without it.
  `wait.go` reads `run.state.ForkSubagents` at the audit (line 1670) and at the `Forked` gate (line 1515), never the attaching spec, and D4 joins `forks` from the record into the attach check.
  A finished run is harvested without the check, but its `finalize` audit still runs, and D7 fails `Wait` naming the engine when the engine lacks the capability.
  Start refuses a `ForkSubagents` spec on an engine without `forks`, and `shuttleengines.New` refuses `forks` without the capability.
  The branch is therefore reachable only through R2's records with no `engine` key, read under a changed default.
  That is a legitimate fail-loud branch and needs no finding.
- **Attach refusal.**
  The step is named and not denied; F1 covers the age window.
  `removeSupersededStrands` runs only on the no-attachable path, so the refusal on a single attachable candidate is never cleared by shuttle itself.
  The record needs the manual step, which D8 gives.
  A finished run is harvested, not refused, which D8 states (F3 covers D4's wording).
- **InterruptSettle.**
  D4 states the behaviour: an error on an unresolvable record, then a Warn log by each caller and the longest settle among the resolved sessions, none when all fail.
  Reading `tenderengine/pause.go` and `orchengine/pause.go`, the orch watcher's compare is `now.Sub(firstKey) < settle` over one session.
  A failed resolution gives zero and one Warn, because the first-key marker is deleted on that tick.
  The next step, the idle probe or the interrupt, resolves the same record and is refused, so no second key goes out.
  Tender's settle sits in a bounded retry loop, where the same refusal follows.
  No finding.
- **Webster's default-engine bound.**
  The conditions are stated: a second registered engine, a Webster spawn spec naming an engine, an `engine:` change across a restart or recover, and R2's records with no `engine` key.
  R4 names the acting tasks.
  Only the placement is flagged, in F4.
