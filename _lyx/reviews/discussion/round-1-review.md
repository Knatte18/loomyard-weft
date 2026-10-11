---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:31
    summary: D1 treats the 5-minute prompt-cache lifetime as a fixed fact, but lyx already sets the TTL per role through claude_prompt_cache_ttl_roles, so the ping interval constant ignores the knob and puts a provider specific outside claudeengine
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:57
    summary: D2's Bound says the probe ends a run only where the boundary gate would pass on expiry, but finalize judges the mechanical entry over a mid-turn tree with no re-prompt, so the probe can halt a run Stuck that the boundary would have re-prompted
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:148
    summary: D11 claims the delivered-at stamp ties the verdict to the writer's SendMessage, but verdict.json records neither who recorded it nor what prompted it, and batten's note is a second route, so the two files do not isolate the hop
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:35
    summary: A failed ping is logged with the gate's pending way-forward, which tells the operator to run lyx loom review notify, a step that re-sends an already delivered notice and spends a delivery prompt
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:27
    summary: D1 does not say whether a ping is carried while the reviewer-liveness hold is in force, and the gate returns early on the hold before any carry branch
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:88
    summary: D5 covers a slug addressed from the prime and no slug in a task worktree, but not a slug addressed from another task worktree, which resolveReviewTarget also accepts
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:138
    summary: D10's in-flight claim names runs whose reject was already let through, but such a run has passed the gate and left Discussion-Write, so it never re-enters the gate on resume
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:46
    summary: An uncapped review spends at most 14 pings before the 60-minute bound, not 15, since expiry is checked before any carry and pings are timed from delivered-at
---

### [BLOCKING] D1 rests on a fixed 5-minute cache lifetime that lyx configures per role

**Location:** `_lyx/discussion/decision-record.md:31` (D1 Decision and Rationale); `_lyx/discussion/support-log.md` Rejected alternatives.

**Issue:**
D1 says "the interval is below the 5-minute prompt-cache lifetime" and makes the interval a constant in `internal/parentreview`.
In this repository the lifetime is not a fixed fact.
`internal/shuttleengine/template.yaml` carries `claude_prompt_cache_ttl: 5m` and `claude_prompt_cache_ttl_roles` (template: `driver: 1h`, `webster: 1h`).
`internal/shuttleengine/claudeengine/doc.go` says the engine sets `CLAUDE_CODE_PROMPT_CACHE_TTL` per role on launch and resume, accepting `5m` or `1h`.
The Discussion-Write strand runs under `discussionRole` (`internal/loomengine/discussion.go:103`), which the template leaves at the 5m default.
Three consequences the record does not address:

1. The existing knob is an alternative for Part 1's goal: one line mapping the discussion role to `1h` keeps the writer warm for the 60-minute wait bound with no ping, no stencil, no `delivery.json` fields and no new gate branch.
   It has a cost of its own: every cache write in the writer's long exploring session is billed at the 1h write price instead of the 5m price.
   The record must weigh pings against it.
   It is in neither D1 nor the support log's Rejected alternatives, so the decision was taken without it.
2. When an operator maps the discussion role to `1h`, the 4-minute pings refresh a cache that was never at risk, which is pure spend; D1 has no rule tying the pings to the role's resolved TTL.
3. The Rationale's prices (read at about a tenth, cold re-write at about 1.25 times) are the 5m-TTL prices; under `1h` the cold re-write price differs, so the payoff arithmetic holds only for the default.

The 4-minute constant also encodes a Claude Code cache property in `internal/parentreview`.
`PATTERN-shuttle-provider-seam` keeps provider specifics under `internal/shuttleengine/claudeengine`, where the TTL is already resolved, so hard-coding its value in the gate package breaches that constraint.

The board entry assumed the 5-minute lifetime too, so the writer inherited the premise, but the record is where the design is fixed, and the knob sits in the same shuttle config D1 relies on.

**Fix:**
Add the role-TTL alternative to the deliberation and decide between pings, a `1h` discussion role, or both, with the cost comparison in D1's Rationale and the losing option in the support log's Rejected alternatives.
If pings stay, state how they relate to the writer's resolved TTL: either no ping when the role's TTL exceeds the wait bound, or the interval told to the gate from the resolved TTL through a seam, not a Claude-specific constant in `internal/parentreview`.

### [MEDIUM] D2's Bound overstates what the probe can end

**Location:** `_lyx/discussion/decision-record.md:57` (D2 Bound); `_lyx/discussion/decision-record.md:181` (Open risks).

**Issue:**
The Bound says "The probe can end a run only where the at-boundary gate would itself pass on expiry".
That does not hold for the mechanical `discussion` entry.
`finalize` (`internal/shuttleengine/wait.go`) calls `evaluateGate(true)` once and never re-prompts.
A mid-turn writer (the Open risks' hand-requested edit, or a ping turn that touched a file) can leave the decision record invalid at the moment the probe fires.
At a boundary that failure re-prompts, since the recipe gives `discussion` `attempts: 3`.
At the probe's finalize it is a failed outcome: `singlellm.go:208-213` and `multillm.go:142` map a failing `Result.Gate` to `Stuck`.
The Discussion-Write row declares no `on_stuck` (`contracts/recipes/loom-recipe.yaml`), so the run halts, with the request already marked expired by `Final`.
The Bound's second sentence notes that the mechanical entry must still pass, and Open risks says it "judges the tree as it stands", but neither says what happens when it does not: the probe cuts off a run the boundary would have recovered.
This is the attack-surface question the rubric asks every new routing edge to answer.

**Fix:**
State the actual reach in D2's Bound: the probe can halt the run `Stuck` when the tree fails the mechanical entry mid-turn, where the boundary would re-prompt.
Then decide, in D2, whether that is accepted (and say so as an Open risk with its consequence), or whether the probe is narrowed, for example by not firing while the mechanical entry fails over the current tree.

### [MEDIUM] D11's proof does not isolate the hop it claims to prove

**Location:** `_lyx/discussion/decision-record.md:148` (D11 Decision and Rationale), and the acceptance line for D11.

**Issue:**
D11 says the round's verdict "must be recorded by the orch's reviewer" and that "the delivered-at stamp is what ties the verdict to the writer's message".
The files cannot show either.
`verdict.json` holds `kind`, `recorded_at`, `consumed` and `superseding` (`internal/parentreview/doc.go`, `store.go` `RecordVerdict`): nothing records who ran the verb or what prompted it.
`delivered_at` is the writer's own stamp that it sent the message, not evidence the orch received it.
D11's Rationale itself names batten's waiting-review note as a second route to the orch, which is the same objection the lens review used to retire the `daemon-wakeups` evidence (support log, item 5).
A verdict recorded after the stamp is explained equally by either route.
This run's own files show the gap: `_lyx/reviews/parent-review/round-1/delivery.json` has `delivered_at` 19:52:27 and `verdict.json` an approve at 19:52:43, and nothing in either says which notice the orch acted on.

**Fix:**
Either name evidence that does isolate the hop (for example the orch's or fork's transcript showing the writer's message as the prompt it acted on, cited in the landing description), or narrow D11's claim to what the two files show and record the remaining gap as an Open risk.

### [LOW] A failed ping points at the wrong way forward

**Location:** `_lyx/discussion/decision-record.md:35` (D1, "a failed send is logged as any pending send is").

**Issue:**
The closures' `pending` helper (`internal/parentreview/gate.go:93`) always sets `SendFailedWayForward` to "run `lyx loom review notify <slug>` to re-send the notice", and `handleGatedBoundary` quotes it in the failed-send Warn.
For a keep-alive ping on a delivered request, that step adds a waiting notify, which makes the gate carry the delivery prompt again (the `WaitingNotifys > 0` branch), re-sending a notice the reviewer already has.

**Fix:**
State in D1 what a failed ping's Warn says: that nothing needs doing and the next ping, if any remains, retries; and that the ping carries its own way-forward rather than the delivery prompt's.

### [LOW] D1 leaves the ping's place against the reviewer-liveness hold open

**Location:** `_lyx/discussion/decision-record.md:27` (D1 Decision).

**Issue:**
`gate()` returns `pending("")` when `holding(r)` reports the reviewer not live (`internal/parentreview/gate.go:183`), before any carry branch.
D1 does not say whether a ping is carried while the hold is in force.
Placing the ping after the hold lets the writer's cache expire during an orch relaunch; placing it before spends pings while the reviewer may have lost the notice.
The plan writer would have to pick.

**Fix:**
Decide in D1 whether the hold suppresses pings, with the one-clause reason.

### [LOW] D5 misses the slug-from-a-task-worktree case

**Location:** `_lyx/discussion/decision-record.md:88` (D5 Decision).

**Issue:**
D5 selects `lyx batten status <slug>` for "a sibling worktree by slug from the prime" and `lyx loom status` for "the task worktree itself".
`resolveReviewTarget` (`internal/loomcli/review.go:70`) resolves a given slug to its sibling worktree from any worktree, so a verb run from one task worktree naming another slug falls under neither clause.

**Fix:**
Key the choice on whether a slug was given, not on where the verb runs, or name the third case.

### [LOW] D10's in-flight claim names the wrong runs

**Location:** `_lyx/discussion/decision-record.md:138`.

**Issue:**
Under pass-at-cap a reject is "let through" when the gate passes on the consumed cap reject, after which the run leaves Discussion-Write for Discussion-Bouncer and never evaluates the gate again; the binary is embedded (`contracts/recipes/recipes.go`), so only a run still inside Discussion-Write reads the new cap.
The claim holds for a run whose cap reject is recorded but whose rewrite has not yet passed the gate: there `RejectedRounds` is 1, below the new cap of 2, so the gate begins round 2.
As worded it promises a second review to runs that already moved on with an unreviewed rewrite.

**Fix:**
Reword to the run whose reject is recorded but whose rewrite has not yet passed the gate, and say that a run already past Discussion-Write keeps its unreviewed rewrite.

### [NIT] The uncapped worst case is 14 pings

**Location:** `_lyx/discussion/decision-record.md:46`; `_lyx/discussion/support-log.md` interview item 3 and the "Uncapped pings" rejected alternative.

**Issue:**
`gate()` checks expiry against `OpenedAt` before any carry, and pings are timed from `delivered_at`, which is never earlier than `opened_at`, so the last ping strictly before the 60-minute bound is the 14th.
The conclusion stands, since 14 cache reads at about a tenth still cost more than one cold re-write at about 1.25.

**Fix:**
Say 14, or "about 15".

## Focus notes

The directive's entries each reached a verdict:

- D1 arithmetic: the quarter-of-a-cold-resume and roughly 16-minute payoff follow from 4 minutes and 3 pings at 5m-TTL prices (three pings at 0.1 against an expected saving of a third of 1.15); the 15 is 14 (F8). The premise under the interval and the prices is F1.
- D2 and D10 attack surface: D10's Bound is accurate. D2's is not (F2), and the Open risks line matches D2's finalize but omits the consequence.
- D2/D3 consistency: the two agree. The probe runs on D3's cadence, the deadline is never extended, the attach case is covered by "whether or not the loop has seen the entry pend", and the acceptance lines match. Not a defect.
- D10 against D7 and the acceptance lines: with `attempts: 2` and no pass-at-cap, `gate()` consumes the first reject and re-prompts (rejected 1, below 2), the rewrite's boundary begins and opens round 2, and the second reject is terminal at the cap; `SupersedeCapReject` then accepts the approve, since round 2's request carries cap 2. Matches. The in-flight claim is F7.
- D8 and D9: D8's Bound states what it skips; D9 adds no verb, flag or bypass. Nothing they need is stranded in the support log. Not a defect.
- Completeness before leanness: each lens disposition that implies a requirement (ping count in `delivery.json`, failed ping counts, probe read-only and unlocked, numbered ping text, no pings before delivery) is in D1, D2 or the acceptance lines. Not a defect.
