---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:151
    summary: An advisor that fails to start on a fresh start is told to a chair that does not exist yet, since D8 starts the chair last, and no delivery path is defined for that notice
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:152
    summary: D7 delivers notices by Run.Send, which needs an idle chair, and D6's own rationale says the chair is mid-turn while drafting; the retry at each poll names a loop seatengine does not have
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:153
    summary: An advisor that reaches done has its strand and run directory removed by shuttle unless KeepPane is set, so the chair cannot reach it again; D5 and D7 say nothing happens, and no Decision sets KeepPane
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:138
    summary: Every seat renders the parent directive, which gives advisors a SendMessage route to the parent; D6's bound covers only peer messages and states nothing about what reaches the parent, against the Goal's channel the parent never sees
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:121
    summary: Still live is undefined; a timed-out or died advisor keeps its strand, and with one Timeout and advisors started first, an advisor timing out before the chair is the routine case, so a settled advisor's strand can outlive the step
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:136
    summary: D6 says shuttleengine carries no provider words and D3 says the channel wording lives only in the two seat blocks, but shuttle's hold notice and the parent directive stencil already name SendMessage
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:231
    summary: The Added after Discussion section restates D6, is not a numbered Decision, and its not meant for this leaves open whether D7's Go-side Send notice is allowed
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:142
    summary: The rationale in D6, D8 and D11 argues against non-chosen options that support-log Rejected alternatives already holds
---

### [BLOCKING] An advisor's failed start is told to a chair that does not exist yet

**Location:** `_lyx/discussion/decision-record.md:151` (D7, second rule), with D8 at line 172 and D9 at line 182.

**Issue:** D7 says an advisor that fails to start leaves the step running and "the chair is told once".
D8 and D9 start advisors in table order and the chair last.
On a fresh start, an advisor fails before the chair has a strand or a handle, so the notice has no recipient.
D7's only stated delivery path is the next rule's Go-side `Send` "on the chair's handle", and that handle exists only after `StartGated` returns.
The plan writer is left to choose between holding the notice until the chair starts, putting it in the chair's prompt through a marker, or sending it after the chair's start.
Each choice behaves differently: a `Send` right after start lands only once the chair's first turn idles (see F2).
The resume path (D8, live chair) is sound: the probe returns the chair's handle before any advisor notice is due.

**Fix:** State in D7 how a notice raised before the chair exists reaches it.
One option is a reserved marker on the chair's prompt that lists the advisors that failed to start, which needs no `Send` and is told exactly once.
Add the matching acceptance criterion.

### [BLOCKING] D7's `Send` notice needs an idle chair, and D6 says the chair is not idle

**Location:** `_lyx/discussion/decision-record.md:152` (D7), against D6's rationale at lines 141–142 and Open risks line 270.

**Issue:** D7 tells the chair about an advisor's death "by a one-line Go-side `Send` on the chair's handle, retried at each poll until it lands or the step ends".
`Run.Send` (`internal/shuttleengine/run.go`, `Send`) waits for an idle session and fails with `ErrSessionBusy` when the session stays busy past the send-ready window.
D6's rationale rejects shuttle's `Send` as the channel because it "needs an idle receiver", and it designs the chair to "keep drafting instead of ending its turn to wait".
So the notice lands only when the chair ends a turn without its outputs, which is a held turn.
That held turn also notifies the parent (D10), or else the chair writes its outputs first and the step ends with the notice never delivered.
Every failed attempt also blocks for the send-ready window.
"At each poll" names no loop: in the burler precedent, the seat engine joins handles by calling each handle's blocking `Wait`, and shuttle's poll loop is internal to `Wait`.
The plan writer cannot tell which goroutine retries, how often, or whether a retry may block the join.

**Fix:** Decide the delivery path in D7 and bound it.
For example: the notice is typed by `Send` from a dedicated goroutine per notice, every retry bounded by the chair's deadline, and the D6 rationale states that the notice reaches the chair only at an idle turn boundary.
Or route advisor-death notices through the chair's prompt-side channel instead.
Name the trade-off the chosen path accepts: a chair that never idles never hears the notice.

### [BLOCKING] An advisor that reaches done loses its strand, so the chair cannot reach it again

**Location:** `_lyx/discussion/decision-record.md:153` (D7, "Advisor reaches done: nothing happens"), with D5 at line 119, D3's advisor block at line 98 and D2's `Seat` fields at line 64.

**Issue:** Shuttle classifies `OutcomeDone` on the first events tick at which every output file exists (`pollEventsTick` in `internal/shuttleengine/wait.go`), not only at a turn end the agent intends as final.
On Done, `finalize` removes the strand and deletes the run directory unless `Spec.KeepPane` is true.
So "nothing happens" is false: an advisor that writes its output disappears from `ListAgents`, and the chair's next `SendMessage` to it goes nowhere.
The advisor block's own wording makes this the expected path: the advisor "writes its output when its task is done".
The chair's dialogue with an advisor therefore ends the moment the advisor writes its file.
That contradicts the Goal's ask-and-answer channel, and D2 neither carries nor rules out `KeepPane`.

**Fix:** Decide the advisor's lifetime past its output in D5.
Either advisors run with `KeepPane` set and are removed by D5's stop on the chair's terminal result, or the advisor block says the output is written last, only on the chair's wrap-up request, and an advisor is unreachable after it.
Add an acceptance criterion pinning the chosen behaviour.

### [MEDIUM] The parent directive gives advisors a route to the parent that D6 does not bound

**Location:** `_lyx/discussion/decision-record.md:138` (D6 bound), with D3's reserved markers at line 91, the advisor block at line 98 and the Goal at line 22.

**Issue:** D3 fills `parent_directive` for every seat, and PATTERN-parent-directive requires every spawned role's top-level stencil to render it.
The directive, `contracts/stencils/parent/parent-directive-parent.md`, tells the agent that its parent is the one to ask, by `SendMessage`.
So every advisor is told how to message `ly:orch`.
The advisor block forbids only "the parent for a design question", which leaves every other question open.
The Goal says the advisors' traffic travels "over a channel the parent never sees".
D6's bound covers advisor-to-advisor messages alone and states nothing about the advisor-to-parent edge this design adds.
Under the attack-surface question, this new routing edge has no stated bound.

**Fix:** State in D6 what an advisor may send the parent and what bounds it, for example "nothing; the advisor block overrides the parent directive and every question goes to the chair".
Also decide which parent directive variant the engine renders for advisor seats.
Reword the Goal if advisors may reach the parent at all.

### [MEDIUM] "Still live" is undefined, and a settled advisor's strand can outlive the step

**Location:** `_lyx/discussion/decision-record.md:121` (D5), with D7 at lines 152 and 155.

**Issue:** D5 stops "every advisor still live" when the chair settles.
`finalize` removes a strand only on Done, so an advisor that timed out or died keeps its strand in reed.
If "live" means "its `Wait` has not returned", those strands are never stopped and outlive the step.
If it means "its strand is tracked", `Stop` on a settled handle is safe: it keeps a terminal record's outcome and removes the strand.
D5 should say which.
One `Timeout` governs every seat, and advisors start before the chair.
An advisor's normal state is a held turn waiting for questions (D7), so an advisor that has not written its output reaches its deadline before the chair does, by construction.
Under D7 every such advisor produces a death notice to the chair near the end of the chair's work.
The record treats this as an edge case, not as the routine path it is.

**Fix:** Define "still live" in D5 as every advisor whose strand reed still tracks, whether its `Wait` has settled.
State in D7 that an advisor timing out before the chair is the routine end of an idle advisor and is told as any other timeout, or decide that advisors' deadlines run to the chair's.

### [LOW] Provider-word and single-source claims are false as written

**Location:** `_lyx/discussion/decision-record.md:136` (D6), line 100 (D3), line 246 (Constraints).

**Issue:** D6 says "`seatengine` and `shuttleengine` carry none" of the provider's words.
`internal/shuttleengine/hold.go`'s `holdNotice` already writes "SendMessage" into the notice it builds.
D3 says the channel wording "lives nowhere else" than the two seat blocks, yet `parent-directive-parent.md`, which every seat renders (F4), carries `SendMessage` wording.
On the focus's premise question: these are claims local to this design and not cross-cutting invariants, and PATTERN-shuttle-provider-seam covers the engine side, so no PATTERN entry is missing.
The claims only need to be true.

**Fix:** Reword D6 to "`seatengine` adds none", and limit D3's single-source claim to seat-to-seat channel wording.

### [LOW] The "Added after Discussion" section duplicates D6 and leaves D7's `Send` ambiguous

**Location:** `_lyx/discussion/decision-record.md:231`.

**Issue:** The section repeats D6's decision, and Scope's Out list already defers to `creel`.
It sits outside the numbered Decisions and breaks the record's `**Decision.**`/`**Rationale.**` shape.
Its "lyx shuttle send exists but is not meant to be used for this" agrees with D6 for seat-to-seat traffic.
It does not say whether the engine's Go-side `Run.Send` notice of D7 counts as "this".
A plan writer reading it literally could drop D7's delivery path.

**Fix:** Fold the operator's decision into D6 and delete the section.
State there that `lyx shuttle send` and `Run.Send` are never the seats' channel, and that D7's engine-to-chair notice is the one Go-side use.

### [LOW] Decisions re-litigate rejected alternatives

**Location:** `_lyx/discussion/decision-record.md:142` (D6 rationale, lines 142–143), line 176 (D8 rationale), line 203 (D11 rationale).

**Issue:** D6's lines on typing through `Send` and on a producer mailbox, D8's "stopped rather than inherited", and D11's "rebuilding it over handles would route every loom LLM row through new code" argue against options that `support-log.md`'s Rejected alternatives already records.
The writer's stencil keeps Decisions to Decision + Rationale and leaves the case against an alternative out.
Under the completeness-before-leanness test, one constraint must stay in the record first: `Run.Send` needs an idle receiver and a single line.
D7's notice path depends on it (F2).

**Fix:** Move that constraint into D7 or Constraints, then cut the argued alternatives from the three rationales.

## Verdicts on focus items with no finding

- D8 against D4's holder-name check: consistent.
  A stopped advisor's strand is removed by `Stop`.
  An advisor whose probe finds nothing has its superseded strand removed by the superseded-removing probe shape D8 names.
  So the name is free when the fresh start forms it, and a holder that survives both is D4's start error.
- D3's include form: blocks are read from the told directory, one level deep, and checked at construction.
  It adds no route past an existing guard, so no bound is missing.
- D10's quiet-hold knob: the Decision states what it skips and that nothing else reads it, so the bound is met.
- D7's advisor-hang rule: it states its bound, the chair's directive, and Open risks names the residual, so the attack-surface question is met.
