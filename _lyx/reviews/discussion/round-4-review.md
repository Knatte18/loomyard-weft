---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 (awaiting with parent_notice), D6, Open risks
    summary: The suppression of batten's notice for an awaiting stop with a parent_notice has no real bound, since a failed or skipped relay is recorded only in a report no notice points to, and the suppression is not tied to the relay reaching the notice's own destination
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7 (chained set-option, concurrent calls), Acceptance criteria, support-log.md Rejected alternatives
    summary: D7 rests on a false premise that a chained set-option on a shared window name fails, since on tmux 3.6 it succeeds on the older window and leaves the new window with remain-on-exit on, so a dead batten window stays
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D7 (strand-window resolution and fallback)
    summary: The resolution rule counts only a live Selvage or strand pane, so a dead Selvage corpse falls to the current-window fallback, and the pane-generation read is routed through ids it exists to validate
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints (PATTERN-cli-cobra and PATTERN-refusal-way-forward), D7 refusals
    summary: The constraint says the batten refusal lands a refusal-spec row, but refusal-spec.md lists batten refusals out of scope and has no batten section, and the second D7 refusal names no way forward
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D4 (since, episode key) against D5 (episode keys)
    summary: D4 keeps since in the notice's episode key and calls the episode rules unchanged, but D5 moves the quiet and api-error keys to agent activity and the marker records only sent notices, so the first-sight time of an unsent notice has no stated home
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D3 (baseline modification time)
    summary: The baseline is stat'ed before the call's status read, but the spawn arm reads the status twice, and a modification-time difference cannot always see a write on a coarse-resolution filesystem while the wait has no time limit left
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D3 (cadence list), D4 (not-queued retry)
    summary: The not-parked resume retry has a cadence but no bound on its log output, and the end of the no-strand notice retry is not stated
---

### [MEDIUM] The bound on an unannounced awaiting stop is not a bound

**Location:** `decision-record.md` D4 ("An `awaiting` child whose status carries a non-empty `parent_notice` gets no state-changed notice…", and its `Bound:` line), D6, Open risks.

**Issue:**
The focus asked whether a dead driver or a failed relay leaves the stop announced by no channel.
A dead driver is covered: batten sends when the driver strand has ended, read at `notice_probe_s`.
A failed relay is not covered.
D4's `Bound:` line says the driver records a failed send in the report, but the report is announced by no notice for this stop, so nobody is pointed at it; it names the hole and does not bound it.
The same hole opens when a live driver skips the relay without a send failing.
Open risks says a driver that does not follow D6 is "noise, not a failure", which is true for an extra message and false for a missing relay on this stop.
Separately, the suppression keys only on `parent_notice` and driver liveness.
D6 writes the watched marker only for a batten in the prime with a recorded orch strand, because only then the driver's parent is the notice's destination.
A batten outside the prime suppresses its notice to the prime's orch while the driver relays to another worktree's orch, so the prime's orch hears nothing, against the closing Open risk that a stop reaches the orch "twice, never not at all".

**Fix:**
State a bound as a mechanism.
Batten suppresses the notice only when this batten wrote the watched marker for the run, so a batten whose destination differs from the driver's parent always sends.
A stop that is still awaiting with no decision record a fixed time after its report exists (a constant, 10 minutes) gets batten's notice even though the driver is alive, so a failed or skipped relay delays the announcement by at most that time and never loses it; a successful relay followed by a slow decision costs one extra notice.
Correct the Open risks entries and add the cases to the acceptance criteria.

### [MEDIUM] D7's failure mode for a shared window name is false

**Location:** `decision-record.md` D7 ("A window whose chained option failed, for example on a name two concurrent calls share…", and the `Bound:` paragraph), Acceptance criteria; `support-log.md` Rejected alternatives and Question ledger Q21.

**Issue:**
Run against tmux 3.6 with `remain-on-exit on` set globally, `new-window -d -P -F '#{window_id}' -n batten:dup … \; set-option -w -t '=sess:=batten:dup' remain-on-exit off` exits 0 when a window of that name already exists.
The option lands on the older window; the new window keeps `remain-on-exit on`.
When the second batten then exits on the run lock, its window stays as a dead pane, which the Decision says cannot happen ("its window closes, or its chained option fails on the shared name and the op kills the window").
The targeting itself is right: on a unique name the option reaches the new window and the strands' window keeps `on`, and a batten that exits at once closes its window.

**Fix:**
Drop the claim that the chained option fails on a shared name.
After the invocation, the op reads `remain-on-exit` back from the new window by the id the invocation printed, and kills the window by that id and refuses when it is not `off`.
Add the shared-name case to the `tmux`-tier acceptance criterion and correct the rejected-alternative and ledger text.

### [LOW] Strand-window resolution counts only live panes and routes the generation read through unvalidated ids

**Location:** `decision-record.md` D7 ("A session with no live Selvage pane resolves…", "While the session has a live Selvage or strand pane…").

**Issue:**
Reed keeps a dead Selvage pane as a corpse in the strands' window (`remain-on-exit on`), and heals it through the same pane listing.
With only a corpse and no live strand pane, the rule falls to the current-window target, which is a batten window when one is selected: the batten pane is then read as untracked and Selvage is recreated into it.
The claim that a batten window is never read as the strands' panes therefore holds only for a live pane, not for a present one.
The Decision also lists the pane-generation read among the ops that go through the seam, but that read exists to validate recorded pane ids, and it only needs session-scoped fields, so resolving a window from those ids first is circular.

**Fix:**
Resolve from the recorded Selvage pane when it is present in the multiplexer, alive or dead, then from any present recorded strand pane, and fall back to the current window only with none; keep the pane-generation read session-scoped and outside the seam.

### [LOW] The batten refusal constraint names a spec row that has no home

**Location:** `decision-record.md` Constraints (`PATTERN-cli-cobra` and `PATTERN-refusal-way-forward`), D7 ("the verb refuses naming the failure").

**Issue:**
`PATTERN-refusal-way-forward` binds the modules with a section in `contracts/specs/refusal-spec.md`, and that file's "Out of scope" lists batten refusals as having no section.
The constraint "lands its spec row" therefore cannot be met without creating a section the spec says a later audit adds.
The refusal for a failed window creation names no way forward.

**Fix:**
Rewrite the constraint: both refusals name a way forward and a test reaches them, and no spec row lands because batten has no section.
Give the failed-creation refusal the way forward `lyx batten run <slug>` in a terminal.

### [LOW] `since` and the episode key disagree between D4 and D5

**Location:** `decision-record.md` D4 ("`since <time>`: … kept in the notice's episode key", "the episode rules … are unchanged"), D5 ("Episode keys move from the status file's modification time to the activity").

**Issue:**
Every notice carries `since`, defined as the first-sight time of the state and kept in the episode key.
Quiet and api-error keys are now activity, so they cannot hold it, and the driver-dead key is unstated.
The marker records only sent episodes, so for a pending notice `since` has no stored home at all, and after a batten restart it is re-read from the file.
Today's key also contains the file's modification time, so "the episode rules are unchanged" is not true of the new `since`.

**Fix:**
State that `since` is held by the wait for as long as the same state is seen, re-read from the file's modification time when batten starts, and is not part of the key of a quiet, api-error or driver-dead notice; a restart that finds the file rewritten in the same state may send one notice again.

### [LOW] D3's baseline ordering is right but not complete

**Location:** `decision-record.md` D3 ("The baseline modification time is stat'ed before the call's status read…").

**Issue:**
The decision does state the ordering the focus asked about.
Two gaps remain.
"The call's status read" is singular, but the spawn arm reads the status again after the spawn, and the baseline must precede the read that defines the state the call began with.
"Is seen" is not guaranteed by a modification-time difference alone: two writes within one timestamp tick (coarse on some filesystems) leave the time equal, and with the 12-hour limit gone, nothing else would ever read the stale state.

**Fix:**
Say the baseline precedes the read that defines the state the call began with, and make the decode also run at each `notice_probe_s` check whatever the modification time says, so a missed write is seen within that interval.

### [LOW] Two retries have no stated log bound or end

**Location:** `decision-record.md` D3 (the retry of a resume refused as not parked), D4 (the no-strand retry).

**Issue:**
Every costly read has a stated cadence.
The not-parked resume retry logs an Info line per attempt today, and nothing says whether that stays one line per attempt for a driver that never parks.
The no-strand retry logs one Warn per episode, but the decision does not say when it stops: the episode ending, a strand being recorded, or the `Done` return.

**Fix:**
Log the not-parked retry once per episode and say it retries until the driver parks or the state changes; state that the no-strand retry ends with its episode or at the `Done` return, where the last attempt is made.
