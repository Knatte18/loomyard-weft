---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: decision-record.md D5
    summary: D5's bound does not cover a stale or unreachable marker, because the marker path is never stated to be per-attempt, removed before a spawn, or re-derivable by a resumed process
  - id: F2
    severity: MEDIUM
    class: decision
    location: decision-record.md D6 and Acceptance criteria (resumed run)
    summary: No Decision states resume behavior when only one half is live, one is gone while the other is done, or the marker already exists
  - id: F3
    severity: MEDIUM
    class: consistency
    location: decision-record.md D1, D6, D7
    summary: D1 does not say the two runs are awaited concurrently though D6 needs that, D7 omits the reviewer gate from its ordering, and the single Result.Gate is unassigned between the two gates
  - id: F4
    severity: MEDIUM
    class: design
    location: decision-record.md D8 and Acceptance criteria (Disputed)
    summary: The Disputed criterion presumes a checker of the fixer-report that does not exist, since only the file's existence is read today
  - id: F5
    severity: LOW
    class: design
    location: decision-record.md D9
    summary: D9 says every entry is parsed and resolved at config load, but LoadConfig only parses grammar and registry resolution happens in ResolveReview
  - id: F6
    severity: LOW
    class: consistency
    location: decision-record.md Acceptance criteria
    summary: Behaviors in D4, D9 and D12 have no matching acceptance criterion (per-half timeout, single friction writer, skills and PATTERN directive on both halves)
  - id: F7
    severity: LOW
    class: decision
    location: decision-record.md D9 and D4
    summary: Whether the fixer's deadline counts the time it spends waiting for the review is left open
  - id: F8
    severity: NIT
    class: consistency
    location: decision-record.md Auto-mode assumptions
    summary: The last Auto-mode assumption restates D8's rationale and the default-provenance bullet restates support-log Q9
---

### [MEDIUM] D5's bound does not cover a stale or unreachable marker

**Location:** `decision-record.md` D5, Constraints (`PATTERN-told-geometry` bullet).
**Issue:** D5 bounds the verb with "only Go writes the marker, and only after D3's checks".
That covers who writes, not which attempt's checks the marker proves.
Today `BurlerProducer` runs up to two attempts per round and archives only the review and fixer-report before each, and a resumed process re-enters `Call` through the live-round probe without calling `Engine.Run`.
The record names a marker under `.lyx/burler/round-*/`, which today is a fresh `MkdirTemp` directory per `Engine.Run`.
That makes a marker from a prior attempt unreachable by accident, but nothing in the record says so, and the Constraints bullet builds the path from that directory.
Under a resume the same property breaks the other way: a fixer still live from the previous process polls the path named in its old prompt, and the new process, which never created that directory, cannot derive it, so it can neither write that marker nor tell whether one exists.
A plan writer who stabilises the path to fix the resume case would reintroduce a stale marker that releases the fixer before the current reviewer's checks passed, which breaks the review-before-fix invariant of `PATTERN-review-round`.
**Fix:** State in D5 that the marker path is told to the engine with the round's other paths and is a pure function of the round's review path, so any process re-derives the same path.
State that the caller removes any existing marker before an attempt spawns either half, in the same step that archives stale outputs, and that Go writes it only for the attempt in flight.
Replace the `PATTERN-told-geometry` constraint's "built from the round directory it already creates" with the told-path wording.

### [MEDIUM] No Decision for resume with a partial or mixed round

**Location:** `decision-record.md` D6; Acceptance criteria, "A resumed run whose round has a live half reattaches to it".
**Issue:** D6 states failure outcomes while both halves are started by one process.
The resume criterion promises reattachment and no second agent per half, but no Decision says what happens when only one half is live, when the reviewer is done and the fixer gone, or when the marker already exists.
D6 plus the criterion do not suffice: the probe today finds a run by output files and respawns when the found run died, and with two strands every mixed state needs a rule the plan writer would otherwise invent.
Today a review without a fixer-report is an incomplete round that is archived and re-run, which gives a consistent default.
**Fix:** Add a resume paragraph to D6.
Both halves live: attach to both, and Go writes the marker when the reviewer's attached run completes with a passing gate.
Exactly one half live, or one gone with the other done: Go stops the live half, removes its strand, archives the round's outputs and the marker, and spawns both halves again as attempt 1.
Bound: no second agent for a half that is live, and no round ends with a half live.

### [MEDIUM] Join semantics, gate ordering and the single Gate field disagree across D1, D6 and D7

**Location:** `decision-record.md` D1, D6, D7.
**Issue:** D1 says both runs start "before waiting on either", while D6 requires that a fixer dying while the reviewer runs stops the reviewer.
A sequential wait on the reviewer then the fixer would not notice the fixer's death until the reviewer ended, which can be the whole round timeout; D1 never says the waits are concurrent.
D7 says verdict and findings are parsed "after both runs reached done and the fixer's gate passed", leaving out the reviewer's review-parse gate, although D5 makes that gate a precondition of the fixer's progress.
D7 also keeps one `Gate` while D3 and D4 put different gate entries on different runs, and `BurlerProducer` reads `Result.Gate` to build a gate-failed `Stuck` and its attempts pointer.
D6 lists "gate exhausted" among reviewer outcomes "anything but done", but an exhausted gate is a done outcome with a failing gate today, and D6 does not say which `Result` fields such a round returns.
**Fix:** State in D1 that Go waits on both runs concurrently and the first terminal event decides the round.
State in D7 that the parse happens after the reviewer's gate passed and the fixer's run reached done with its gate passed, and that `Result.Gate` carries the failing half's outcome, otherwise the fixer's.
State in D6 that a reviewer whose review gate is exhausted returns outcome done with that failing gate, no verdict or findings and the fixer stopped, which is the shape `BurlerProducer` already maps to a gate-failed `Stuck`.

### [MEDIUM] The Disputed acceptance criterion presumes a checker that does not exist

**Location:** `decision-record.md` D8; Acceptance criteria, "The fixer-report's `Disputed` section is accepted by the round...".
**Issue:** The criterion says a disputed finding is "not counted as unfixed by anything that checks the report".
Nothing parses the fixer-report today: the round reads its existence for completion (`roundComplete`, the shuttle's output-file wait), hands it to the next round's reviewer as prior context, and the Bouncer judge stencils do not read its sections.
The criterion therefore points a plan writer at a consumer to find or invent, and a parser would be new surface beyond the task.
D8's own text is consistent: "the existing deferred section keeps its meaning" holds because the fixer stencil is the only definition of that section.
**Fix:** State in D8 that no Go code reads the fixer-report's sections, so a Disputed section changes the round's completion not at all and reaches the next round only through the existing hydration.
Reword the criterion to "a fixer-report with a Disputed section completes the round exactly as one without, and the fixer stencil names the section, its evidence requirement and its factual-premise bound".

### [LOW] D9 places resolution at config load

**Location:** `decision-record.md` D9, Acceptance criteria (`review` and `fix`).
**Issue:** D9 says every entry is "parsed and resolved at config load".
`LoadConfig` validates model-spec grammar only; registry resolution happens in `loomengine.ResolveReview`, called when the run is wired.
The criterion's "unparseable entry fails config load" is accurate, while the premise that unknown models fail at load is not.
No consumer of the scalar `review` key or of `review_timeout_min` beyond `ResolveReview` and the `shedrecipe.Env` fields it fills was found, so "loads unchanged" itself holds.
**Fix:** Say in D9 that entries are parsed at config load and every entry of both lists is resolved in `ResolveReview` at run start, so a bad later-round entry fails before round 1.

### [LOW] Decisions without a matching acceptance criterion

**Location:** `decision-record.md` Acceptance criteria.
**Issue:** D9's "`review_timeout_min` stays the deadline of each half's run", D4's "the friction directive goes to the fixer only" and D12's "both halves load the same skills and receive the PATTERN directive" state behavior with no criterion.
**Fix:** Add one criterion per behavior, each as a row or assertion in an existing test per `PATTERN-test-economy`: each half's spec carries the round timeout, only the fixer's prompt carries the friction directive, and both specs carry the skill list and the PATTERN directive.

### [LOW] The fixer's deadline versus its waiting time is left open

**Location:** `decision-record.md` D9 ("`review_timeout_min` stays the deadline of each half's run"), D4.
**Issue:** Both halves start together under the same timeout, so the fixer's wait for the review counts against its own deadline.
A long review leaves the fixer less time than the reviewer had, and a fixer timeout is a round failure under D6.
The record does not say whether that is intended.
**Fix:** State in D9 that each half's deadline starts at its own spawn and the fixer's waiting counts against it, with the reviewer's own deadline bounding the wait.

### [NIT] Auto-mode assumptions restate Decision rationale

**Location:** `decision-record.md` Auto-mode assumptions.
**Issue:** The assumption "A disputed finding is acceptable to the operator's 'grain of salt' only with evidence (D8)" repeats D8's rationale, and the default-provenance assumption repeats the Q9 entry in `support-log.md`.
No requirement in either is missing from the Decisions, so nothing needs extracting first.
**Fix:** Drop the D8 restatement and keep the defaults' provenance only in the support log, leaving the assumptions that decide plan behavior.

## Focus departures

None.
