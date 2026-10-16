---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7 (visit-time rule)
    summary: D7's visit-time rule counts halt, pause and awaiting wait time into a row's time, except when a goto intervenes, where the same wait is dropped; the decision states neither
    fix: In D7, after the sentence on goto entries, state how wall time spent while the run is not running is attributed. Either say plainly that a row's time is wall time between history stamps and so includes any halt, pause or awaiting wait that preceded the call (and that the interval before a goto entry is dropped), or define the visit's start as the later of the previous entry's at and the resume, naming where that resume time is read from. Add an acceptance criterion pinning the chosen behavior for a row resumed after a failed call and for a row reached by goto after a halt.
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D7 (box vocabulary and the running row's clock)
    summary: D7's four boxes and its running row clock assume a running run; the box and clock of the current row while the run is paused, blocked or awaiting are not stated
    fix: In D7, state which box the current producer's row shows when the run's state is paused, blocked or awaiting (either add boxes for them or map each to one of the four), and whether its current visit's time keeps running to now or stops at the last entry's at while the run is halted. Extend the checklist acceptance criterion with one halted state.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D8 (append trigger and growth bound)
    summary: D8's once-per-notice append holds only within one batten process; a held-back or dropped notice is composed and attempted again after a batten restart or a repeated row failure, giving a second log line, which contradicts the exactly-one-line acceptance criterion and the stated growth bound
    fix: "In D8, define a notice's identity for the append: either key the append on the notice's episode key (notice.go's noticeKey for the Run-Shed notices, the row marker for row-failure notices) recorded in a marker written at append time, so a restarted batten or a repeated failure of the same episode appends nothing, or state that each batten process logs its own first attempt and that a notice still undelivered when batten restarts is logged again. Align the growth-bound sentence and the acceptance criterion's exactly-one-line wording with the choice."
---

### [MEDIUM] D7's row time silently includes halt waits, and drops them before a goto

**Location:** `_lyx/discussion/decision-record.md` D7, the paragraph beginning "A history entry's `at` is stamped when its producer call returns".

**Issue:** The stated stamping matches the code: `internal/shedengine/run.go` builds the entry in `appendHistory` with `At: nowRFC3339()` after `def.Producer.Call` returns; `appendOrFold` keeps the first call's `At` and moves `LastAt` on each fold; `goto.go` appends `{Producer: target, Outcome: goto, At: now}` and admits only a halted run; and a call with an empty outcome records nothing.
The acceptance criterion for a row reviewed twice is satisfiable: the loom ring Review → Fix → Bouncer → Review gives two separate entries for the review row, summed.
The gap is what the rule does with time the run spent not running.
The status file carries no resume stamp (the step 3b resume write in `run.go` stamps nothing), so a visit measured from the previous entry's `at` includes every wait before the call: a run halted overnight after a failed call, a pause at a boundary, or an `awaiting` stop all land in the next visited row's time.
When a goto intervenes the same wait is dropped instead, because the goto's `at` starts the next visit, and D7's own clause that a no-outcome call "adds its time to the next recorded visit" then no longer holds, since that time falls before the goto.
The checklist would show, for example, a review row "taking" ten hours that the run spent failed, while the same halt followed by a goto shows nothing.
Either reading may be acceptable, but the plan writer gets no rule for it, and two implementations of D7 as written would disagree.

### [LOW] D7's boxes and running clock do not cover a halted run

**Location:** `_lyx/discussion/decision-record.md` D7, "a box (done, running, failed, not reached)" and "the running row's current visit from the last entry's `at` to now".

**Issue:** `internal/shedengine/status.go` has six states: running, paused, done, blocked, failed, awaiting.
D2 keeps halted runs on screen, and a pair's checklist stays up through every halt, but D7 names a box only for running and failed.
A run paused at its next row (after `pause` or a `goto`), blocked on its bounce budget, or awaiting the operator has a current producer whose box and clock D7 leaves open: the plan writer must guess whether a blocked review row shows as failed, running or not reached, and whether its time keeps counting to now.

### [MEDIUM] D8's "once" is per batten process, not per notice

**Location:** `_lyx/discussion/decision-record.md` D8, first paragraph and "a notice goes once per condition episode, so it grows by a few lines per run"; acceptance criterion "A batten notice appends exactly one line … also when it is sent while no orch strand is recorded".

**Issue:** Which notices reach the log: every notice batten composes, which excludes those never composed (`Notify` nil, `PairDir` unresolved, a state change not yet ready, an episode key already kept), and includes those held back or dropped, as D8 says.
The trigger "at its first delivery attempt" is implementable for the Run-Shed notices as the moment `deliver` in `internal/battenshed/notice.go` creates the `noticeDelivery` record, and for row notices as the call into `send` in `rownotice.go`.
Both records are in-process only.
`p.episode.deliveries` lives in the producer's memory and starts empty in a new batten process, and the episode marker (`saveEpisode`) records a key only once `deliver` reports it queued.
`rowNotifier.failed` writes its marker only when `send` returns true.
So a notice held back for a missing orch strand, or dropped after `noticeMaxAttempts`, is composed and given a fresh first attempt by the next batten process for the same episode, or, for a row notice, by the next failure at the same history length, and appends a second line.
The criterion's "exactly one line … also when it is sent while no orch strand is recorded" then holds only within one process, and the growth bound's premise that a notice goes once per episode holds only for delivered notices.
The log stays bounded by operator restarts, so this is no blocker, but the decision must say which identity the append is keyed on.

## Focus items judged not defective

- D1 and D3 when strands end: after `lyx orch stop` the prime window holds the `runs` strand (top, content) and Selvage, and D3's rule that with no sharing and no positional strand live the bottom-most strand takes the free rows assigns every row; a pair window whose positional strands have all ended reduces to the same case with `loom-status`. Selvage alone falls to `planCells`' existing sole-band shape in `render/rules.go`.
- D3 anchor and kind pairs: the anchor vocabulary in `render/types.go` is below-parent, own-window and hidden, plus the new top; D3's table gives every pair a height or a refusal, own-window stays refused, and the remainder rows of the sharing split go to the bottom-most sharing strand, which matches today's bottom-most-expands shape in `height.go` and leaves the positional-only output untouched.
