---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:181
    summary: "D6's bound that nothing lands a round with an open BLOCKING finding under `unattended` is false: D2 keeps the arbiter's `hard-cap`, `circling` and `stalemate` settles as today, and at a hard cap the arbiter accepts, so a round with open BLOCKING findings lands unattended, including the round after D6's one continue."
    fix: "In D6, extend the unattended rule to every escalation the arbiter may accept: under `unattended` the arbiter hands on any escalation, of any cause, whose round holds an open BLOCKING finding or a red gate, and the orch pauses and holds it as D6 says. Name in D6 how the arbiter learns the level (a field of `arbiterstop.Stop` or its prompt), change D2's hard-cap row to say so, and add to D6's acceptance criterion a hard-cap escalation with an open BLOCKING finding under `unattended` that is handed on, not accepted."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:170
    summary: "D6 has the arbiter accept a round with deferred open BLOCKING findings under `false`, where today `arbiterstop.Eligibility` refuses `budget` and the stop reaches the operator; this contradicts D1's `false: no autonomy, as today` and widens admission at `false` with no stated bound."
    fix: "In D6, limit the arbiter's budget accept to `true`; under `false` the arbiter may continue once but hands the escalation on instead of accepting. State in D6's Bound what changes at `false`, if anything, and change D6's acceptance criterion from `accepted under false and true` to `accepted under true, handed on under false and unattended`."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:285
    summary: "Under `true`, a dobby orch with its slack button off drops the dobby's severe post (outbound.go marks it handled), D7 forbids the orch to relay a `settle: operator` halt, and D12 warns only under `unattended`; the severe halt reaches no one while the state record holds it as posted."
    fix: "Widen D12 to both autonomy levels: under `true` or `unattended`, with the dobby's orch's button off or no operator configured, the dobby warns its orch once per episode. Add to D7 that, while the dobby's orch has its button off, the orch relays a `settle: operator` halt in its pane as today. Update the D7 and D12 acceptance criteria to cover `true`."
  - id: F4
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:248
    summary: "D10 chases only a stop record reading `settle: orch`, but the two stalls its own rationale names leave none: an agent question an orch never answers (D11 suppresses the idle notice for it), and a dead driver past its revive cap (D3: no stop at a row; D2: no operator cell). Both still wait until morning."
    fix: "Extend D10: under `true` or `unattended` the dobby also chases an open agent question in creel unanswered after `beam.halt_chase_min`, and a dead driver past today's revive cap, with the same one chase and, under `unattended`, one severe post. Give D2's dead-driver row an operator cell for that case, and add both cases to D10's acceptance criterion."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:77
    summary: "D2's table has no column for `false`, yet D2 returns a verdict per level and its criterion pins `every class at each level`; whether D4's lym settles, D6's arbiter rules and D7's `settle:` stamp apply at `false` is undefined, against D1's `false: as today`."
    fix: "Add to D2 a statement of the verdict at `false`: either lym's column applies at every level and the orch's columns read as today's relay to the operator, or the table returns `operator` at `false` for every class that reaches the orch. State in D4 whether its rows settle at `false`, and keep D2's criterion naming `false`."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:89
    summary: "The rows for stale build, `driver_not_parked`, dead driver or API error, and permission dialog have no operator cell, so under D2's verdict order a third halt at the row never becomes severe, against today's class-5 rule and the Auto-mode assumption that severe classes stay as today under `true`."
    fix: "Give each such row's operator column `third halt at the row`, or state in D2 that the class is no stop at a row and why a repeat of it cannot recur at one row unseen; keep the D2 criterion covering count three for each row."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:92
    summary: "D2 says no other package decides who settles a stop class and gives scope change, cancel and no-settling-action rows operator verdicts, while D7 has the orch classify and relay a scope change under `true` and a halt with no settling action itself; no driver stop carries those classes, and D7's list omits the cancel."
    fix: "In D2, mark the rows the driver cannot detect (agent question, entry overtaken, scope change, cancel, no settling action open, step only the operator can take, provider permission dialog) as orch-classified rows the table lists for the role text, never stamped by lym, and reword `No other package decides` to `No other Go package decides`. Make D7's relay list name exactly those rows, the cancel under `true` included."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:106
    summary: "D3's count is not defined on the records that exist: a handed-on escalation leaves an Awaiting entry, which the class-5 rule does not count; settle records are keyed per stop instance with no class; tackler resumes live in tackler attempt directories; and crash-wake re-steps of an interrupted row are restep records that D3 would count, though D3 says a dead driver counts nothing."
    fix: "In D3, state which entries count: name a handed-on escalation as an Awaiting entry that counts once, say that a settle record gains the class and the halt entry it settled, say that a tackler resume is counted from its tackler attempt, and exclude an interrupted-reinvoke re-step after a driver wake. Add those four cases to D3's tier0 criterion."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:174
    summary: "D6's `once per review segment` needs a memory that the arbiter already continued, which no record holds since each arbiter is fresh and `Eligibility` is stateless; it also conflicts with D3's count over the row's whole history, and D6 leaves undefined a missing or stale fixer report and the arbiter's right to run `lym board upsert`."
    fix: "In D6, name the record the arbiter reads to know it continued this segment (for example the segment's circling decision files of cause `budget`), state that this per-segment count, not D3's history count, bounds the continue, define a missing or older-than-the-round fixer report as `not deferred`, and state that the arbiter's settings allow `lym board upsert` with kind note or that the note is filed by Go after the accept."
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:138
    summary: "D4's pause resume rests on a line that does not exist: the orch pause watcher posts `pauserecord.ResumeLine` only to the orch's own interrupted session, and `queueResumeLine` exists nowhere; the dobby's existing `liftPending` pass, given only to drivable runs, is not named, and how a hand-back is matched to its pause entry is unstated."
    fix: "Rewrite D4's pause paragraph from the code: state that today a paused run gets no pass until a hand resume (`internal/dobbyengine/doc.go`), that the dobby already records a lift in `liftPending`, and that D4 extends that lift to resume a run whose last stop is the pause hand-back, matched by the pause entry's scope; drop the resume-line sentence and the Notes line naming `queueResumeLine`."
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:139
    summary: "D9 and D6 have the unattended orch hold a run by pausing it, but D4 resumes every run whose last stop was a pause hand-back once that pause lifts, so a blanket `lym tender resume` auto-resumes runs held for the operator, after D10 has already ended their chase."
    fix: "In D4, exclude from the lift resume a run whose pause the orch set as a hold (D6, D9), for example by a hold reason the pause entry carries, and add that case to D4's pause criterion."
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:159
    summary: "Batten's pass runs `lym loom start` as a child and retries on its not-parked refusal; D5 makes `lym loom start` wait up to the keep-warm cap, which for a strandless child is the default await cap, so a dobby pass blocks in the child, against D5's claim that no dobby or batten tick blocks."
    fix: "In D5, state that the wait applies only when an agent strand calls the verb, or that batten's child start passes a no-wait option, and that the batten child keeps today's refusal; add that case to D5's criterion."
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:158
    summary: "D5 keeps the run-lock refusal on a non-running run at once, but that branch is the driver finishing its post-run work and refuses with the same `driver_not_parked` kind and `again in a few seconds` text the orch looped on, so the loop the board entry names survives for it."
    fix: "In D5, make the run-lock-held branch on a non-running run wait for the lock's release under the same cap and wake rule, or state why it cannot; add it to D5's criterion."
  - id: F14
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:189
    summary: "D7 writes `class:`, `settle:` and `halts_at_row:` into `the run's stop record, the one the dobby's child read carries`, but `ChildReading` reads only the child status's state, producer, error and history; the record names no file for the stamps and says nothing of what clears them on the next transition, which D10 depends on."
    fix: "In D7, name where the three facts are written (for example the halt's history entry in the child status, which `ChildReading` already reads) and state that the next transition supersedes them; D10's `still halted at the same stop` then reads the same entry."
  - id: F15
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:91
    summary: "The permission-dialog row and D9 give the orch a dialog to read and answer, but nothing notices one: the support log says no mid-run dialog is detected, autonomous runs skip permissions, and `claudeengine` reads a dialog as not idle, so the quiet notice never fires; how a dialog reaches the orch is left unresearched."
    fix: "Settle the fact in the record: either state, citing `internal/shuttleengine/claudeengine/command.go` and `session.go`, that autonomous sessions run with permissions skipped, so the row and D9's dialog rule are dropped, or name the existing signal that brings a dialog to the orch."
  - id: F16
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:213
    summary: "D8's mention dedup keys on the slug within `slack.remind_after_min`, so a second, distinct severe halt of the same run inside that window (such as D10's post after a D7 post, or a new halt after a resume) notifies no phone."
    fix: "In D8, key the dedup on the slug and the halt (its row and history entry, or the halt's id in the post), so a relay and a turn reply for one halt notify once and a new halt notifies again; add a distinct second halt to D8's criterion."
  - id: F17
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:275
    summary: "D11 has every runner's held notice read the open question through the existing probe, but `creelengine.Store.AwaitsReply` also answers true while the strand holds an unread reply, so the held notice is suppressed in exactly the case D11 says is no wait."
    fix: "In D11, state that the held notice and the quiet notice read a probe that answers only an open question of the strand's own, and that a held unread reply is read separately and named; cite `creelengine.Store.AwaitsReply` as the probe to split."
  - id: F18
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/discussion/decision-record.md:34
    summary: "The study's `Hung command or gate` class (3 cases, 18 to 40 min, two after the operator pushed; its table proposes lym ends it past its time limit) is neither a row of D2's table nor named in Scope, Out; D11 only names the command in a notice."
    fix: "Either add a D2 row for a hung command or gate with its verdicts, or add it to Scope, Out with the reason, such as the judgment the study's section 2 assigns it."
  - id: F19
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:344
    summary: "Several acceptance criteria name a test's intent but not its tier, leaving the tier for the reviewer to infer: D3's second bullet, D4's second and third, D6's second, D8's second, D10's first, D11's second and third, and D12."
    fix: "Give each of those bullets its tier, as tier0, tier1 or `the lowest that fits` with the tier the behavior needs, the way the D1 and D5 bullets do."
  - id: F20
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:200
    summary: "Role-autonomy has the orch file a repeating stop's bug with `lym selfreport create` before it relays it; once lym stamps such a stop `settle: operator` and the orch never relays it, nothing in D7 or D9 says who files the self-report."
    fix: "In D7, say that the orch still files the self-report for a `settle: operator` halt that is a third halt at a row, without relaying it, and say so in the role-text Review criterion."
  - id: F21
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:251
    summary: "Under `unattended` the orch relays a halt with no settling action open itself, leaving the run halted with `settle: orch`, and one chase interval later D10 posts the same halt as severe, so the operator gets two severe posts for one halt."
    fix: "In D10, end the chase once the orch has relayed the halt as severe (for example on a `[operator] severe halt <slug>` post from the orch for that slug after the halt), or drop the orch's own relay of that class under `unattended` in D7."
  - id: F22
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:90
    summary: "The dead-driver row's lym bound is `within today's caps`, while D2 says every lym rule takes its bound from the table."
    fix: "Name the bound in the row, or state in D2 that the revive caps stay where they are and the row only records them."
  - id: F23
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:133
    summary: "Only the board-claim row says how its class is told apart without message text; the rows for launch failure, session timeout, fixer fork failure, lost strand and gate attempts exhausted name no field that identifies them, while today's rule table matches only kind, transient class and two reason prefixes."
    fix: "In D4, state for each new row that its class is raised as a named kind or field where the stop starts, as the board-claim bullet does, so no rule matches on message text."
  - id: F24
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:388
    summary: "The plan-writer note says the settle records are written in `internal/sheddriver/tacklerrecords.go`; that file holds tackler attempts, and settle records are written by `AppendSettleRecord` in `internal/sheddriver/settle.go`."
    fix: "Change the note to name `internal/sheddriver/settle.go` for the settle records, and `tacklerrecords.go` for the tackler attempts D3 counts."
  - id: F25
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:329
    summary: "Open risks says `push-retry-not-halt` is at Parent-Align; the board shows it at Code-Burler-Fix."
    fix: "Drop the row name from the risk, keeping only that this task's runs start after `push-retry-not-halt` lands."
---

### [BLOCKING] The arbiter's hard-cap accept lands open BLOCKING findings under `unattended`

**Location:** `_lym/discussion/decision-record.md:181`
**Issue:** D6's bound holds only for cause `budget`. D2 keeps `hard-cap`, `circling` and `stalemate` with the arbiter "as today", and `arbiterstop.Eligibility` admits them with no level check (`internal/arbiterstop/arbiterstop.go:117`). At a hard cap `circling continue` refuses (`internal/loomcli/circling.go:127`), so accept is the only open verb, and the study records 17 arbiter accepts there. The interactions, walkthrough and bound-breaker lenses each found this.

### [BLOCKING] D6 changes level `false`

**Location:** `_lym/discussion/decision-record.md:170`
**Issue:** D6 has the arbiter accept deferred open BLOCKING findings under `false`, which today reach the operator. D1 defines `false` as "as today", and D6 states no bound for the change. The interactions lens and the handler both found this.

### [BLOCKING] A severe halt can reach no one under `true`

**Location:** `_lym/discussion/decision-record.md:285`
**Issue:** `postMessages` in `internal/slackengine/outbound.go` drops and marks handled the mail of an orch whose button is off. D7 bars the orch from relaying a `settle: operator` halt, and D12 warns only under `unattended`. The walkthrough and bound-breaker lenses both found this.

### [BLOCKING] D10's chase misses the stalls its rationale names

**Location:** `_lym/discussion/decision-record.md:248`
**Issue:** An unanswered agent question and a dead driver past its cap leave no `settle: orch` stop record. D11 also silences the idle notice for the open question. The walkthrough and bound-breaker lenses both found this.

### [MEDIUM] The table defines no verdict at `false`

**Location:** `_lym/discussion/decision-record.md:77`
**Issue:** The table's columns cover only `true` and `unattended`, yet D2 and its criterion speak of every level. The interactions lens and the handler both found this.

### [MEDIUM] Rows with no operator cell lose today's repeating-stop rule

**Location:** `_lym/discussion/decision-record.md:89`
**Issue:** Role-autonomy class 5 makes any third halt at a row severe. These rows never reach `operator`.

### [MEDIUM] D2 and D7 disagree on who classifies the orch-only classes

**Location:** `_lym/discussion/decision-record.md:92`
**Issue:** The driver cannot detect a scope change, a cancel or "no settling action open". D2's exclusivity claim and its operator cells conflict with D7's orch relay list.

### [MEDIUM] D3's halt count does not map onto the records that exist

**Location:** `_lym/discussion/decision-record.md:106`
**Issue:** The count is undefined on today's records. `StopKey` carries `HistoryLength` (`internal/sheddriver/stopkey.go`), `repairCount` matches exact keys (`internal/sheddriver/driverules.go:237`), and Awaiting entries are not halts (`internal/reportengine/timingwaits.go:88`). Interrupted-reinvoke re-steps after a wake are restep records (`internal/sheddriver/drive.go:809`).

### [MEDIUM] D6's once-per-segment continue has no memory

**Location:** `_lym/discussion/decision-record.md:174`
**Issue:** Each arbiter is a fresh spawn, `Eligibility` is stateless, and `arbiterstop.Stop` carries no level and no continue count.

### [MEDIUM] D4's pause resume rests on a line that does not exist

**Location:** `_lym/discussion/decision-record.md:138`
**Issue:** `pauserecord.ResumeLine` (`internal/pauserecord/record.go:31`) is the interrupted-session line. `queueResumeLine` has no match in `internal/`. The dobby's lift pass (`triggerFor`, `liftPending` in `internal/dobbyengine/passes.go`) is not named.

### [MEDIUM] The orch's hold-by-pause is undone by D4's lift resume

**Location:** `_lym/discussion/decision-record.md:139`
**Issue:** A run held by the orch's pause carries a pause hand-back as its last stop, so a blanket `lym tender resume` would have D4 resume it.

### [MEDIUM] Batten's child `lym loom start` would block a dobby pass

**Location:** `_lym/discussion/decision-record.md:159`
**Issue:** `childSpawnError` in `internal/battencli/wire.go` retries on the not-parked refusal, but D5 exempts only batten's resume, not its child start.

### [MEDIUM] The run-lock branch keeps the refusal the orch looped on

**Location:** `_lym/discussion/decision-record.md:158`
**Issue:** `internal/loomcli/resume.go` refuses a held run lock on a non-running run with `shedrun.StartNotParkedKind` and a retry text.

### [MEDIUM] D7's stop record has no named home

**Location:** `_lym/discussion/decision-record.md:189`
**Issue:** `ChildReading` in `internal/dobbyengine/runnotice.go` reads only the child status's state, producer, error and history.

### [MEDIUM] Nothing notices a permission dialog

**Location:** `_lym/discussion/decision-record.md:91`
**Issue:** The support log records that no dialog is detected. `claudeengine` reads a dialog as not idle, and autonomous runs skip permissions. The row's premise is a deferred research question. The walkthrough and bound-breaker lenses and the handler all found this.

### [MEDIUM] The mention dedup silences a second, distinct severe halt

**Location:** `_lym/discussion/decision-record.md:213`
**Issue:** The window keys on the slug alone, not on the halt. The walkthrough and bound-breaker lenses both found this.

### [MEDIUM] The existing reply probe also answers for an unread reply

**Location:** `_lym/discussion/decision-record.md:275`
**Issue:** `creelengine.Store.AwaitsReply` (`internal/creelengine/verbs.go:1082`) is true while the strand holds an unread reply to one of its messages. I checked this.

### [MEDIUM] The study's hung-command class is dropped

**Location:** `_lym/discussion/decision-record.md:34`
**Issue:** Section 1 of the study counts the class, and section 3 gives it a row. The record does not place it in the table or in Scope, Out.

### [MEDIUM] Test tiers are missing from several criteria

**Location:** `_lym/discussion/decision-record.md:344`
**Issue:** The discussion spec requires each new test's intent and tier under `## Acceptance criteria`.

### [LOW] No one files a repeating stop's self-report

**Location:** `_lym/discussion/decision-record.md:200`
**Issue:** The role's self-report step is tied to the relay, which D7 removes for `settle: operator` halts.

### [LOW] One halt gets two severe posts under `unattended`

**Location:** `_lym/discussion/decision-record.md:251`
**Issue:** The orch's own relay leaves the stamp `settle: orch`, so D10's later post repeats it.

### [LOW] The dead-driver row's bound is not in the table

**Location:** `_lym/discussion/decision-record.md:90`
**Issue:** "Within today's caps" names no bound, against D2's own rule that bounds come from the table.

### [LOW] The new driver rows name no identifying field

**Location:** `_lym/discussion/decision-record.md:133`
**Issue:** Only the board-claim bullet says how its class is raised. `decideDrive` reads kind, transient class and two reason prefixes.

### [LOW] The plan-writer note names the wrong file for settle records

**Location:** `_lym/discussion/decision-record.md:388`
**Issue:** `SettleRecord` and `AppendSettleRecord` are in `internal/sheddriver/settle.go`.

### [LOW] Open risks names a stale row

**Location:** `_lym/discussion/decision-record.md:329`
**Issue:** `lym board list` shows `push-retry-not-halt` at Code-Burler-Fix.

## Rejected

- bound-breaker F11, the relay's escaping of a mention: where the mention is inserted relative to `slackEscape` is an implementation detail Plan settles, not a design gap.
- walkthrough F6, the gate-attempts re-step is weak: D4 follows the study's proposed table ("one fresh spawn"), which the board entry says to design from; it is a choice, not a defect.
- walkthrough F5's arbiter-input part is merged into F1 and F9; its `lym board upsert` part is in F9.
- bound-breaker F4's cross-class counting example is merged into F8.
