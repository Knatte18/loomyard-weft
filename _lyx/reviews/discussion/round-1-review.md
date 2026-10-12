---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:101
    summary: D7's per-row time rule rests on a false premise, since a shed history entry's at field is stamped when the producer call returns, so it marks a visit's end and not its start, and the visit count ignores folded and goto entries
    fix: In D7, replace the parenthetical time rule with one built on end times, a row visit lasting from the previous history entry's at (or the seed's started_at for the first entry) to its own at, and the running row from the last entry's at to now. State that an entry's repeats count as further visits of that row, that goto entries are not visits and add no time, and that a call which records no history entry (an error with no outcome) adds its time to the next recorded visit. Extend the checklist acceptance criterion with a row whose visits include a folded repeat and a goto.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:35
    summary: D1 keeps the runs strand after lyx orch stop removes the orch strand, leaving a window of one top content strand and Selvage, and D3 names no strand that takes the rows the content strand's maximum leaves free, so the layout cannot sum to the window height
    fix: In D3, add the rule for a window where no positional or sharing strand is live, for example that the bottom-most strand of the window, content strands included, expands past its maximum and draws no summary line, or that the content maximum is lifted for the last strand of the window. Add an acceptance criterion that after lyx orch stop the prime window lays out the runs strand and Selvage with every row assigned.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:110
    summary: D8 appends only notices queued or logged for the orch, once per delivered notice, but batten holds a notice back when no orch strand is recorded and drops it after its attempt cap, so whether a held-back or dropped notice reaches the log is unstated, and D1 expects runs while the orch is stopped
    fix: In D8, replace queued or logged for the orch, once per delivered notice with the exact trigger, for example once per notice batten composes, at its first attempt, whether it is then queued, held back for a missing orch strand or dropped after its attempts. Extend the notice-log acceptance criterion with a notice sent while no orch strand is recorded.
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:53
    summary: D3 adds top to the anchor vocabulary and three size kinds but defines heights only for top plus content and below-parent plus positional or sharing, leaving a top positional strand, a top sharing strand and the even split's remainder rows without a stated outcome
    fix: In D3, state for each anchor and kind pair either its height rule or that Rules refuses it with an error, as it refuses own-window, and state which sharing strand takes the remainder rows of the even split (for example the bottom-most), so a mixed window has one derivable height per pane.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:42
    summary: D2 and D7 leave three reachable states without an outcome, a prime batten run directory with a seed but no batten status file, a readable seed with no started_at (WriteSeed admits one from before the field), and a checklist whose seed names a recipe shedbuild cannot load
    fix: In D2, state whether a run with no batten status file is in flight, and that a seed without started_at sorts with the runs that have no readable seed. In D7, state what the checklist pane shows when the seed or its recipe cannot be read, for example the state line followed by recipe unreadable.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:160
    summary: The lyx cleanup risk is a constraint stated as a risk and names test-tmp-cleanup, which has landed on main (bc9ee7a02) and whose batten segment empties directories under shed's scratch root, which D8's path through internal/shedrun may land in
    fix: Remove the Open risks bullet and add a Constraints bullet stating that the hub notice log's path lies outside every directory lyx cleanup sweeps (shed's records and scratch roots, .scratch and the lyx temp root), so no cleanup segment removes or treats it as a run, and name that the branch must be rebased onto a main that contains lyx cleanup.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:51
    summary: D2's departure paragraph and A5's reed-multi-window narrative are deliberation whose requirements already sit in D2's line content, D8, Scope Out and D3, so they belong in support-log.md
    fix: Delete D2's last paragraph (This departs from the board entry's note...), since D2's line list, D8's attach command and A2 carry its requirement; move its reasoning to support-log.md under Rejected alternatives beside Full attach command on each overview line. Cut A5 to shared strands stack in rows (D3) and move the reed-multi-window and depth-note narrative to the support log's Interview.
  - id: F8
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:162
    summary: Several testable decision behaviors have no acceptance criterion, the seed start-time ordering with no-seed runs last by slug, a run in Worktree-Create shown before its pair exists, lyx orch stop leaving the runs strand, and history entries naming a removed row listed after the recipe rows
    fix: Add acceptance criteria for those four behaviors, D2's ordering and Worktree-Create rows, D1's orch stop, and D7's removed-row entries, as rows of the existing overview, orch and checklist criteria where one fits.
---

### [BLOCKING] D7 measures row time from the wrong end of a history entry

**Location:** `_lyx/discussion/decision-record.md:101` (D7)

**Issue:** D7 computes a row's time as "a history entry's start to the next entry's start".
`internal/shedengine/run.go` appends the entry after `def.Producer.Call(ctx)` returns, with `At: nowRFC3339()`, so `at` is the visit's end time.
Read literally, the rule credits each row with the next row's duration and drops the first row's time.
`HistoryEntry` also folds identical budget-exempt stuck calls into one entry with `repeats` and `last_at`, and `goto` appends entries with outcome `goto` at the goto's time (`internal/shedengine/goto.go`), so a visit count of one per entry miscounts both.
A call that errors with no outcome records no entry (the skip in `appendHistory`), and its time is unassigned.
The acceptance criterion "the summed time and the visit count for a row reviewed twice" cannot be met by the stated rule.

### [BLOCKING] No strand absorbs the free rows once the orch stops

**Location:** `_lyx/discussion/decision-record.md:35` (D1), with D3 at line 53

**Issue:** `internal/orchcli/stop.go` removes the orch strand, and D1 keeps the `runs` strand on stop, so the prime's strands' window then holds one top content strand and Selvage.
D3 caps a content strand at its content rows, makes Selvage fixed-height, and gives the remaining rows only to sharing strands or the bottom-most positional strand.
With neither live, D3 assigns those rows to nothing, while a tmux layout must cover the whole window.
The same state arises in a pair window if every positional strand ends before the checklist.
The plan writer cannot derive the outcome, and the two plausible readings (stretch the content strand, or refuse) differ in what the operator sees.

### [MEDIUM] The notice log's trigger misses held-back and dropped notices

**Location:** `_lyx/discussion/decision-record.md:110` (D8)

**Issue:** `internal/battenshed/notice.go` and `rownotice.go` hold a notice back with a Warn when no orch strand is recorded, and drop it after `noticeMaxAttempts`.
D8's "queued or logged for the orch" and "once per delivered notice" do not say whether those notices are appended.
D1 keeps the overview useful while the orch is stopped and D2 sends an operator outside the session to the log for the attach command, which is exactly when notices are held back, so excluding them defeats the log's purpose, and including them contradicts "delivered".
This is also the attack-surface check on D8's unconditional append path: its bound on growth (one line per episode) holds, but its trigger is not stated.

### [MEDIUM] Anchor and kind combinations without a height

**Location:** `_lyx/discussion/decision-record.md:53` (D3)

**Issue:** D3 states that top-anchored strands stack above the below-parent stack, and that sharing and positional strands keep one below-parent stack, but `render.Display` carries anchor and size independently.
A top strand that declares no size, or declares sharing, has no stated height, and Rules neither refuses it nor places it.
The even split of rows among sharing strands leaves a remainder when the rows do not divide, and D3 does not say who takes it, though the positional rule's byte-identical output is a Constraint and the render tests pin exact heights.
The mixed window the focus asks about (a top content strand, sharing strands and collapsed positional strands) is otherwise derivable: content takes `min(content, max(min, floor(share × free)))`, collapsed strands take `collapsed_rows`, and the sharing strands split the rest.

### [LOW] States with no stated outcome in D2 and D7

**Location:** `_lyx/discussion/decision-record.md:42` (D2), and D7 at line 97

**Issue:** D2 decides in-flight from the batten status state, but `state.ReadJSONStrict` reports a missing file as not found, and D2's two-poll rule covers read and decode failures only, so a seeded run before its first status write has no outcome.
`internal/shedrun`'s doc says a seed written before `started_at` existed has none, and D2 orders by that field.
D7 reads rows from the seed's recipe through `shedbuild`; a seed or recipe that fails to load leaves the pane's content unstated.

### [LOW] The lyx cleanup risk is a constraint, and its premise moved

**Location:** `_lyx/discussion/decision-record.md:160` (Open risks)

**Issue:** "must not sweep the hub notice log" is a requirement on D8's path, not a risk, and Plan-Write reads it only as a risk.
`test-tmp-cleanup` is no longer on the board; it landed as `bc9ee7a02` on `main`, which this branch does not yet contain.
That commit's `lyx batten cleanup` builds its slug list from the names under shed's records and scratch roots and empties orphan scratch, so a log declared through `internal/shedrun` under those roots would be read as a slug.

### [LOW] Deliberation narrative in the decision record

**Location:** `_lyx/discussion/decision-record.md:51` (D2), and A5 at line 150

**Issue:** Completeness-before-leanness test applied first: D2's departure paragraph carries one requirement (lines carry no attach command; the log does), which D2's line list, D8 and A2 already state.
The remainder is the reasoning behind a rejected alternative, which the support log already records under Rejected alternatives.
A5's requirement (rows, not windows) sits in Scope Out and D3, and its remaining text is board history.
Moving both loses nothing Plan-Write needs.
The Open risks entry on cramped seat rows is a stated risk and stays.

### [LOW] Decisions with testable behavior and no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md:162` (Acceptance criteria)

**Issue:** D2's ordering rule and its Worktree-Create branch of the in-flight rule, D1's "`lyx orch stop` leaves it", and D7's trailing rows for history entries naming a removed row are stated behaviors with no criterion.
Every other Decision's testable behavior maps to a criterion: D3 to the render and seat-table criteria, D4 to the content-rows operation criterion, D8 to the notice-log criterion, D9 to the Alt-key criterion and D10 to the shell-line criterion.

## Focus verdicts not raised as findings

- D4: bounded; the refusals on strand, declaration, pane id and count hold against the acceptance criterion, and the share and minimum cap an oversized count.
- D9: bounded; the steps stay gated on `@lyx_strands` (`internal/reedengine/bindings.go`), and the Alt+z condition gains only the Selvage marker, so a batten window's pane stays outside both.
- D10: bounded; the window ends only on a 0 attach or on the pair directory gone within the wait, and D10 names the `reed down` and reap cases that end with 0.
- D1, D5, D6, D7 ownership: consistent. The orch start adds and replaces `runs`, and nothing removes it; the loom bootstrap keeps its keep, add and replace logic (`resolveStatusStrandAction`, which replaces a live strand from another build, as D1 says), and D7's dropping of the llm arm's `removeStatusStrands` is a layout choice with its rationale stated, not a guard.
