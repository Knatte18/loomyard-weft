---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:29-70
    summary: D1's signal, a hand-on verdict in the stop report, also matches the verb-concluded hand-ons that D2 lists as no-verdict faults, so the D1/D2 split rests on a false premise
    fix: In D1, define the signal as an arbiter attempt with verdict hand-on and an empty cause (the agent's own verdict), and hand the fork the attempt's envelope.json path rather than verdict.yaml. In D2, replace the list item an arbiter refusal, timeout or malformed verdict with two cases, an arbiter refusal (no envelope) and a hand-on carrying a cause (timeout, died, missing, unparseable, precondition), and route both to the fault rule. Drop D1's fallback to the brief's question when the verdict has none, since an agent hand-on requires a question and a verb hand-on is a D2 fault. Update the second Acceptance criteria bullet to name the empty-cause signal, and add a scenario that a hand-on with a cause is treated as a fault.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:33-38
    summary: D1's hand-on fork records decisions and resumes the run past an arbiter that has already handed on, and the Decision does not state what the fork may let through or what bounds it
    fix: Add a Bound paragraph to D1 stating that the fork may run only the brief's decide options (circling accept or continue) and lyx loom decision add, then lyx loom resume. It settles only when a recorded operator decision answers the verdict's question directly. It never settles a question whose answer widens the task's scope, and it never accepts over an open design-wrong finding unless a recorded operator decision covers that finding. In every other case it records nothing and reports back. Mirror the scope and design-wrong limits in the brief's settle condition under the escalation-brief bullets of D1.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:72-78
    summary: D3's joint peer fork and its late-peer fresh fork have no stated relation to D1's per-slug hand-on fork and D2's fault rule, and no bound on what they may record or resume
    fix: In D3, state that each peer escalation is first classified by D1 and D2. A peer escalation that is a D2 fault is investigated as a stop. Peer hand-on escalations go to the one joint fork under D1's settle rule and bound, with each slug named in the orch note as an open fork. For a late peer whose first fork is still open, state that the fresh fork starts only after the first fork's report has arrived and passed the re-check, so it reads a finished outcome.
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:142
    summary: D7's plan-time condition on moving the issue-673 workaround leaves open what happens when reed-add-parentless-placement lands after plan time but before this run lands, and Open risks assigns the other order to no one
    fix: In D7, replace when CLAUDE.md still holds it at plan time with a rule that holds through landing. If the parent's CLAUDE.md no longer holds the issue-673 workaround at Merge-In, ORCHESTRATOR.md carries no issue-673 entry, and the Merge-In resolution drops it. Add that case to the last Acceptance criteria bullet. Reword the last Open risks bullet so it names the follow-up concretely, as a board note on reed-add-parentless-placement stating that its workaround removal must target ORCHESTRATOR.md when this run lands first. Drop the (running) status from D7's rationale, since it goes stale.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:125
    summary: D6's Bound states that the file cannot override the role's rules as a fact, while only a sentence of role text the orch must obey enforces it, and the bound omits that any landed task branch can change the inlined text
    fix: Reword the last clause of D6's Bound to say that the file can carry any text into the orch's role file at every delivery, that it reaches the prime through any landing, including one the alignment agent lands with no orch turn, and that the only guard against a contradicting rule is the role's own section text, which tells the orch to follow the role and raise the conflict to the operator.
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:70
    summary: D2's rationale sentence on the operator doubting the case occurs is deliberation narrative that belongs in support-log.md
    fix: Move the sentence The operator doubts the case occurs, so the rule stays short into support-log.md's Turn 3. In D2's Decision, add a requirement that the fallback rule stays to one short paragraph in the role section, so the plan writer keeps the length constraint.
---

### [BLOCKING] D1's hand-on signal also catches D2's fault cases

**Location:** `_lyx/discussion/decision-record.md:29-70` (D1 and D2)

**Issue:**
D1 routes every review escalation whose stop report names an arbiter verdict of `hand-on` to the settling fork.
D2 treats an arbiter timeout or malformed verdict as an escalation with no `hand-on` verdict.
The code says otherwise.
`internal/loomcli/arbiter.go` `handOn` writes an envelope with `Verdict: hand-on`, a non-empty `Cause` and the synthetic question "The arbiter could not settle this stop (<cause>); decide it yourself …" for every verb-concluded case: `timeout`, `died`, `missing`, `unparseable` (the malformed verdict) and `precondition` (an unsound `resume`).
`internal/loomengine/arbiter.go:193` says `Cause` "is empty for the agent's own verdict and names the failure … otherwise".
The Go driver's stop report (`internal/sheddriver/drivehandback.go`) prints `verdict hand-on, cause <cause>`, and the LLM driver stencil (`shed-template-driver.md:121-122`) takes the hand-on path for these cases too.
So under D1 as written, an arbiter timeout goes to the settling fork, which is told to "decide it yourself", the opposite of D2's fault rule.
The verdict path D1 hands the fork, `verdict.yaml`, may also not exist in these cases, because the agent left none.
The only distinguishing field is the empty versus non-empty `cause`.
D1's fallback for a verdict with no question covers no real case: the arbiter stencil requires `question` on a `hand-on`, and the verb's hand-ons always set one.
The Acceptance criteria encode the same wrong split.

### [MEDIUM] D1's settling fork has no stated bound

**Location:** `_lyx/discussion/decision-record.md:33-38` (D1)

**Issue:**
The rubric's attack-surface question applies.
D1 gives the fork the power to record decisions and resume, and through the brief's `decide_options` to run `lyx loom circling accept`, which takes work with an open design-wrong finding.
The arbiter (`loom-template-arbiter.md`, `## When you hand on`) hands on when the call needs an operator decision that the board entry, decision record and support log do not record, when it widens the task's scope, or on any doubt.
D1's fork reads those same sources plus the operator's board notes.
So the fork settles either from the board notes alone or by reading the same records differently from the arbiter.
The only limit D1 states is that the fork records nothing when the recorded decisions do not answer.
It does not say what the fork may let through: an accept over a design-wrong finding, or a scope-widening answer the arbiter refused to give.

### [MEDIUM] D3's peer forks are not tied to D1 and D2

**Location:** `_lyx/discussion/decision-record.md:72-78` (D3)

**Issue:**
D3 sends both peer escalations to one fork that reads both decision records, and a late peer to a fresh fork that reads the first fork's outcome.
It does not say whether a peer escalation is first classified under D1 (hand-on to the settling fork) or D2 (fault to `## Investigating a stop`).
It does not say whether the joint fork follows D1's settle rule, or what it may record or resume.
It does not say what the fresh fork reads while the first fork is still open and has no outcome yet.
D1's open-fork rule is per slug, so a joint fork spanning two slugs is also left out of the orch-note bookkeeping.

### [MEDIUM] The #673 move condition misses a post-plan landing

**Location:** `_lyx/discussion/decision-record.md:142`, with Open risks at line 182 and Acceptance criteria at line 201

**Issue:**
The board shows `reed-add-parentless-placement` at `running · Plan-Write`, and its scope removes the #673 workaround from `CLAUDE.md`.
D7 moves the workaround only "when `CLAUDE.md` still holds it at plan time".
If the reed run lands after this run's Plan-Write but before this run lands, this run's Merge-In takes a parent `CLAUDE.md` without the workaround, and nothing tells it to drop the copy already moved into `ORCHESTRATOR.md`.
That leaves a workaround for a fixed bug in the orch's prompt.
The three places agree on the plan-time case only.
For the opposite order, Open risks says the other run's removal "must target `ORCHESTRATOR.md`", but no Decision or action gives anyone that job.
"(running)" in D7's rationale is a status that goes stale.

### [LOW] D6's Bound overstates the no-override guarantee

**Location:** `_lyx/discussion/decision-record.md:125` (D6)

**Issue:**
The Bound says the file "can add orch rules but not override the role's".
Nothing mechanical enforces that: the render inlines any text with no size cap, and the only guard is the new section's sentence telling the orch to follow the role and raise a contradiction.
The Bound should state what the file can let through and what stops a contradicting rule.
Any task branch can change the file, and the change reaches the orch after a landing, including an alignment-approved one with no orch turn.
Open risks names that route, but the Decision's Bound does not.

### [NIT] Deliberation narrative in D2's rationale

**Location:** `_lyx/discussion/decision-record.md:70` (D2)

**Issue:**
"The operator doubts the case occurs, so the rule stays short" records why the rule was kept brief, which is deliberation.
The plan writer needs only the length constraint, so that goes into the Decision and the narrative goes to the support log.

## Rejected

- D5 against D7 on the Production exception: no inconsistency. The role's `update-plugins.sh` lines are the empty-`deploy` fallback, which D5 drops as dormant here. The orch's own-fix exception was never in the role and moves from `CLAUDE.md` to `ORCHESTRATOR.md` under D7. No Decision edits this task's own board entry.
- D4: `lyx loom reject --help` confirms the rework rows D4 names (PR-Rework in loom, Darn-Write in darn).
- D6's render sites: `RenderRoleFile` has exactly the three callers the plan-writer notes name, and the `command_index` refusal D6 models exists.
- D7's premise that no spawned role's stencil runs a parentless `lyx reed add`: grep of `contracts/stencils` finds no `reed add` at all.
