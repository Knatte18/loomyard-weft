---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Open risks, RemoveStrand cascade)
    summary: The cascade risk misstates reed's RemoveStrand and leaves the recursive flag open; pin a non-recursive removal as a Decision instead
  - id: F2
    severity: LOW
    location: _lyx/discussion/decision-record.md (Decisions, sidecar delete; Acceptance criteria)
    summary: Deleting the status sidecar on the llm arm is YAGNI by the support log's own admission and widens the driver branch past the driving surface
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md (Decisions, invariant amendment)
    summary: The amendment rewrites only the headline and does not say why the first sub-bullet (no code path gates behaviour) still holds
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md (Goal)
    summary: The Goal's never and exactly one overstate what the Decisions deliver before the next start on this build
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md (Acceptance criteria, step does not read the driver)
    summary: The step-does-not-read-the-driver criterion is a structural property no behavioural test checks, and duplicates the whatever-the-driver criterion
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md (Acceptance criteria)
    summary: No criterion covers a failed reed Up now that Up runs outside ensureStatusStrand on both arms of start and in step
  - id: F7
    severity: NIT
    location: _lyx/discussion/decision-record.md (Decisions, ensureStatusStrand keeps its name and body)
    summary: A helper-boundary decision the record itself declares non-binding; its behavioural content is already in the start and step decisions
---

### [MEDIUM] The cascade risk misstates RemoveStrand and leaves the flag open

**Location:** `decision-record.md`, `## Open risks`, third bullet.

**Issue:** The risk says "Reed's `RemoveStrand` cascades through descendants."
It does so only when called with `recursive=true`.
`removeStrandLocked` (`internal/reedengine/strand.go`) refuses a non-leaf with `strand has children, use --recursive` when `recursive` is false, and removes nothing.
The record never says which form the llm arm uses, so the plan writer could pick the recursive one and turn a hypothetical child into a kill.
On the focus question: a live ly-drive strand cannot sit beneath the status strand.
`driverspec.go` builds the driver strand with `Parent: ""`, and `reedDriverPaneProbe.RemoveDriverStrand` already documents the driver strand as parentless and childless.
So the danger the focus names does not exist in today's tree, but only an unpinned flag keeps it from arising later.

**Fix:** Add a Decision: the llm arm removes each status strand non-recursively, so a status strand with children is refused by reed and handled by the warn-and-continue path, never cascaded.
Replace the Open risks bullet with the residual risk that follows from that: a status strand some older build parented anything beneath stays up, with a warning.

### [LOW] Sidecar delete earns no place in the decision record

**Location:** `decision-record.md`, `## Decisions` (sidecar bullet) and `## Acceptance criteria` (second bullet's "and deletes the status sidecar").

**Issue:** The support log's own rejected-alternative entry concedes a leftover sidecar is harmless: `resolveStatusStrandAction` returns `statusStrandAdd` whenever no strand carries the name, and ignores the sidecar in that case.
Only the go arm reads or writes the sidecar.
The delete adds a second driver-gated side effect, a failure path, and a test, to tidy a file nothing misreads.
It is also the one piece of the llm arm that is not "driving surface" selection, so it is what strains the invariant amendment (F3).
Nothing in it is a requirement the plan writer needs, so cutting it loses nothing.

**Fix:** Drop the sidecar Decision and the sidecar clause from the acceptance criterion.
In `support-log.md`, flip the pick: record "delete the sidecar on removal" as a rejected alternative under YAGNI, and update interview turn 7 and ledger Q7 to match.

### [LOW] The invariant amendment is silent on the first sub-bullet

**Location:** `decision-record.md`, `## Decisions`, last bullet; `## Constraints`, first bullet.

**Issue:** `CONSTRAINTS.md`'s first sub-bullet says "no code path gates *behaviour* on it".
The record amends only the headline clause.
Read in context, the sub-bullet bars readers other than the bootstrap verb's own branch, and the amended headline permits that branch to pick the session's strand set.
The llm arm's strand removal therefore fits once the sidecar delete is dropped (F2).
But the record does not say this, and a plan writer or later reviewer reading the unamended sub-bullet will ask the same question this round's focus asks.

**Fix:** In the amendment Decision, state that the first sub-bullet stays unchanged: it governs every code path other than the bootstrap verb's branch, and removing the status strand is that branch choosing the strand set, not a gate on behaviour.

### [LOW] The Goal overstates "never" and "exactly one"

**Location:** `decision-record.md`, `## Goal`.

**Issue:** The Goal says an llm run "never" shows the `loom-status` strand.
The record's own Open risks accept that a running ly-drive session keeps a leftover band until its next `start`, and that a go run driven by bare `step` shows no status strand.
The Decisions deliver the Goal from the first `start` on this build onward, not unconditionally.
The Goal should change, not the Decisions: both accepted gaps are reasoned and cheap.
The Scope Out stray ly-drive strand on a go run does not contradict "exactly one", since seeds are immutable per run and a go run never spawns that strand.

**Fix:** Qualify the Goal: once `lyx loom start` has run on this build, the session shows exactly one driving surface.

### [LOW] "step does not read the seed's driver" is not a behavioural criterion

**Location:** `decision-record.md`, `## Acceptance criteria`, tenth bullet.

**Issue:** No test observes whether a function reads a value it then discards.
The tripwire test only proves the read stays inside `internal/loomcli`, which `step` already is.
The checkable form of this property is already the seventh bullet: step adds no status strand "whatever the seed's driver".
The ordering claim in the second bullet ("before the driver-spawn step") is checkable, by the call order the fake reed and fake driver starter record in `start`'s existing seam tests.

**Fix:** Delete the tenth bullet and make the seventh explicit: `step` behaves identically on a `go`-seeded and an `llm`-seeded run.
The driver-blindness itself stays stated in the `step` Decision, where it belongs.

### [LOW] No criterion for a failed reed Up

**Location:** `decision-record.md`, `## Acceptance criteria`.

**Issue:** `Up` moves out of `ensureStatusStrand` to run unconditionally.
Today a failed `Up` fails `start` with an error envelope and releases the bootstrap lock, and fails `step` with the bootstrap refusal kind.
The criteria check that `Up` runs on both arms, but not that its failure still stops `start` on the llm arm before the watchdog and driver spawn, which is the arm whose error path is new.

**Fix:** Add a criterion: a failed reed `Up` fails `start` on either arm with an error envelope before the watchdog and driver spawn, and fails `step` with the bootstrap refusal kind.

### [NIT] The ensureStatusStrand helper-boundary Decision is plan detail

**Location:** `decision-record.md`, `## Decisions`, the "`ensureStatusStrand` keeps its name and body" bullet.

**Issue:** The bullet fixes a helper's name and body, then says the plan writer may restructure helper boundaries anyway.
Its one requirement, `Up` running outside the strand work so both arms and `step` share it, is already stated by the `start` split Decision and the `step` Decision.
The deliberation is already in support-log interview turn 5.

**Fix:** Cut the bullet from `decision-record.md`.

## Focus directives settled without a finding

- **Step path, two callers:** both claims hold.
  `ensureStatusStrand` has exactly two callers, `start`'s `RunE` and `loomPreStep`.
  The go driver's `lyx loom run` enters through `loomPreRun`, which does no strand work.
  Reed `Up` never launches strands (its doc comment in `lifecycle.go`), and the llm arm's `RemoveStrand` persists the pruned state, so a later reed resume cannot resurrect the removed strand.
  No other verb or daemon path re-adds it.
- **Notes for the plan writer:** file and helper names without `file:line`, signatures or interface shapes sit within what the writer stencil allows, and the optional section is explicitly a non-exhaustive head start.
  Its one requirement, rewording `start`'s long help, already appears in Constraints.
  Not a defect.
