---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:52
    summary: D2 numbers rounds by review-row visits on the false premise that each review visit writes the next Burler round, but BurlerReviewProducer.Call hands on without opening a round, so the checklist's round n and Burler's round-n files diverge
    fix: In D2, replace the two sentences at lines 52-53 with this rule. A review-row entry belongs to the round whose review file its output names, so a hand-on entry (the highest complete round still unjudged, or a restart over an accepted review, both returning a BudgetExempt Stuck without spawning) joins the round already open and opens none. Round n holds the review-row entries of round n, the Bouncer entry that bounced into the first of them, and the fix entries after them. A goto or reroute into the review row opens a new round only when that visit runs a review. Map an entry's output to its round number through the package that owns the round file names (`shedadapters`), never by a second parse in the checklist. Add an Acceptance criterion that a hand-on review entry renders inside the round it hands on, with no new round heading.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:54
    summary: D2 puts every Bouncer visit that routes nowhere, a halt or an awaiting escalation included, as the block's last line outside any round, while D3 shows the escalation and its arbiter attempt in their round and the run goes on after a continue
    fix: In D2, rewrite line 54 as follows. A Bouncer entry that routes nowhere (awaiting, or a halt) stands as the last line of the round in progress, followed by any arbiter line D3 places after it; when no round has opened yet, it stands before round 1. Only a Bouncer Done that ends the segment stands as the block's last line, outside any round. Add an Acceptance criterion that a round-3 escalation followed by an arbiter continue renders the awaiting line and the arbiter line inside round 3, with round 4 after them.
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:149
    summary: D11 keys its advisor rewrite on discussion_producer within one document, but MigrateConfig also runs over local files, sparse worktree files and run_settings (PrepareRunSettings migrates before it validates), which set discussion_advisors without the guard key, so a string list there is silently emptied or misread instead of refused as line 153 claims
    fix: Add to D11 that the `discussion_producer`-keyed rewrites apply only to a document that sets `discussion_producer` itself. A document that sets `discussion_advisors` as string entries without `discussion_producer` (a local file, a sparse worktree file, or a task's `run_settings`, which `configreg.PrepareRunSettings` passes through `MigrateConfig` before validating) keeps those entries unmigrated, so the load refuses them with the way forward at line 153. Restate line 153 so the refusal is said to come after migration, and to cover worktree-layer files too. Extend the migration Acceptance criterion with a run_settings document and a sparse worktree file that each hold a string `discussion_advisors` list and no `discussion_producer`, and are each refused, not emptied.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:105
    summary: With discussion_producer retired, a non-empty discussion_advisors starts advisor seats on its own, so a local override or run_settings can now start any number of advisors that only a guard-layer seats setting could start before; D7 states no such reach or bound, and D8 line 120 calls the change no wider
    fix: In D7, replace the line-105 clause about strand names with a statement of the new reach and its bound. With `discussion_producer` gone, a non-empty list starts advisors in any layer, so a local override or a task's `run_settings` can now add advisor seats. Each seat is named and checked at load, and its model resolves through the registry. It skips no gate, and its cost falls on the run that sets it. Keep the key category none, which is the recommended choice. If the count needs a bound, state it in D7 as a refused maximum. In D8, limit the line-120 bounds-no-more sentence to the lens forks, since `discussion_producer` also kept advisors dormant.
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:104
    summary: No Decision sets the template defaults of the changed keys; internal/loomengine/template.yaml ships discussion_advisors ["sonnet[high]"], inert today under the default single producer, which would start an advisor on every hub whose file lacks the key, and the defaults of discussion_lenses and plan_lenses are unstated
    fix: Add to D7 that the template default of `discussion_advisors` becomes the empty list. Add to D8 and D9 that the template defaults of `discussion_lenses` and `plan_lenses` are empty, matching today's `single` defaults. Add an Acceptance criterion that a `loom.yaml` that sets none of the three keys loads with no advisors and no lens forks.
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:96
    summary: D6 checks only that the formed role passes the role grammar, so a later seat named 2 forms discussion-2, which agentname.MatchesRole reads as reed's numbered form of the lead role discussion; D7 line 106 instead checks the name itself, so the two Decisions state different rules
    fix: In D6's Validate list, require each later seat's name itself to pass the agent-name role grammar ([a-z] first), as well as its formed role, so no formed role can read as a numbered form of the lead's role. Make D7's line-106 check cite the same rule. Add a numeric seat name to the Validate refusal Acceptance criterion.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:8
    summary: Goal 2 says advisor-<n> is never shown, but D11 line 148 migrates seats advisors to the names advisor-<n>, which then show as discussion-advisor-<n> strands until the operator renames them
    fix: In Goal item 2, add the following exception after "are never shown". A `discussion_advisors` entry migrated by D11 carries `advisor-<n>` until the operator renames it.
  - id: F8
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:65
    summary: D3 reads attempts through loomengine's existing readers, but ListArbiterAttempts fails the whole listing on one unreadable attempt.json, and an attempt's number and stop key live inside that file, so the rule at line 64 cannot be met and an unreadable attempt has no stated place
    fix: In D3, replace line 65 with the following. The checklist reads each attempt directory on its own (`loomengine.ReadArbiterAttempt` per subdirectory of `shedrun.ArbiterDir`), read-only and without a lock, so one bad attempt leaves the others drawn. In line 64, show an unreadable attempt as `arbiter attempt unreadable (<directory name>)` at the end of the segment's block, since its number and stop key are unknown. Make the unreadable-attempt Acceptance criterion use that text and place it.
  - id: F9
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:63
    summary: D3 shows an attempt with no verdict as running, so an attempt whose arbiter died or passed its recorded Deadline without a verdict reads running forever
    fix: In D3, line 63, show an attempt with no verdict as `running` only while its `Deadline` lies ahead, and as `no verdict` after it. Add that row to the arbiter Acceptance criterion.
  - id: F10
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:87
    summary: D5 relies on the pause record without stating its bound; pauserecord.Update drops entries resumed more than Retention ago, so a visit that overlapped such a pause counts it as row time again
    fix: Add a sentence to D5 saying that a pause resumed more than `pauserecord.Retention` before the frame is drawn has left the record, so its span counts as row time. Add a pointer to that sentence under Auto-mode assumptions.
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:120
    summary: D8 states the guard bound of discussion_lenses as the forks being turned on only hub-wide or in a worktree's own config, but the guard category bars a key that switches off or hollows a check, so the bound that matters is that no local override or run_settings can turn the lens forks off or change their fan
    fix: In D8, line 120, replace "it can turn lens forks on only hub-wide or in a worktree's own config, never through a local override or `run_settings`" with "only the hub-wide file or a worktree's own config sets it, so no local override or `run_settings` can turn the lens forks off or change their fan".
---

### [BLOCKING] Rounds counted by review-row visits do not match Burler's rounds

**Location:** `_lyx/discussion/decision-record.md:52`
**Issue:** D2 says each review visit writes the next round, as `ResolveRound` numbers them.
`ResolveRound` counts review files on disk, not visits.
`BurlerReviewProducer.Call` (`internal/shedadapters/burlerreview.go`) returns a routine BudgetExempt Stuck without spawning in two cases: the highest complete round is still unjudged, or a restart finds the round's review already accepted.
Each case is a review-row history entry that opens no round.
Under D2's rule, the checklist would show round k+1 where Burler's files still say round k.
The goto criterion has the same two readings, since a goto into the review row can land in a hand-on.

### [MEDIUM] Bouncer entries that route nowhere are placed last, against D3

**Location:** `_lyx/discussion/decision-record.md:54`
**Issue:** An `awaiting` outcome routes nowhere (`internal/shedengine/producer.go`), so line 54 puts the escalating Bouncer entry last and outside any round.
D3's title and the board entry want the escalation and the arbiter decision shown in the round they decided.
After a `continue`, the run goes on, so the line is not last either.
A halted Bouncer that is later resumed or reached again by a goto hits the same contradiction.

### [BLOCKING] D11's migration misreads the documents that lack discussion_producer

**Location:** `_lyx/discussion/decision-record.md:149`
**Issue:** `configsync.reconcileSparseFile` runs `Migrate` over every local file and sparse worktree file.
`configreg.PrepareRunSettings` runs it over a task's `run_settings` before `ValidateLocal`.
`discussion_producer` is a guard key, so local files and run settings never carry it, and a sparse worktree file often does not either.
Read literally, line 149 ("any other `discussion_producer`") empties a string `discussion_advisors` list in those documents without a word, so the operator's advisors silently vanish.
The other reading leaves the strings in place.
Line 153 claims the refusal, but the record never says which reading holds.
The focus asked for exactly this kind of path, where an old value is silently misread.

### [MEDIUM] Advisors become startable from local layers and run_settings with no stated bound

**Location:** `_lyx/discussion/decision-record.md:105`
**Issue:** Today `discussion_advisors` is category none, but it is read only under `discussion_producer: seats`, a guard key (`internal/loomengine/config.go`, `ConfigSpec`).
D8 retires that switch, so any layer, `run_settings` included, can now start advisor seats, each a session that costs money.
D7 explains the category by strand naming and states neither this reach nor any bound.
D8's claim that the lens key "bounds no more than the key it replaces" overlooks that the retired key also kept advisors dormant.

### [MEDIUM] Template defaults of the new and changed keys are undecided

**Location:** `_lyx/discussion/decision-record.md:104`
**Issue:** `internal/loomengine/template.yaml:5` ships `discussion_advisors: ["sonnet[high]"]`, which is inert only because the default producer is `single`.
Once advisors run whenever the list is non-empty, keeping a non-empty default starts an advisor wherever the template supplies the key.
Nothing in D7, D8, D9 or D11 decides that default or the defaults of `discussion_lenses` and `plan_lenses`.

### [LOW] A numeric seat name collides with the lead's numbered form

**Location:** `_lyx/discussion/decision-record.md:96`
**Issue:** `agentname.MatchesRole` treats `<base>-<n>` (n ≥ 2) as the numbered form of `<base>`.
D6 checks only the formed role, and `discussion-2` passes the grammar.
Before this change the lead was `discussion-chair`, so the clash could not happen.
With the lead at the bare prefix, an advisor named `2` reads as a second lead.
D7 line 106 checks the name itself, which would refuse `2`, so the two Decisions disagree.

### [LOW] Goal 2 contradicts D11's migrated names

**Location:** `_lyx/discussion/decision-record.md:8`
**Issue:** D11 line 148 names migrated advisors `advisor-<n>`, which Goal 2 says are never shown.

### [MEDIUM] D3's reader cannot produce its own unreadable-attempt line

**Location:** `_lyx/discussion/decision-record.md:65`
**Issue:** `loomengine.ListArbiterAttempts` (`internal/loomengine/arbiter.go`) fails the whole listing on any subdirectory whose `attempt.json` cannot be read.
`Number` and `StopKey` come from that same file.
So the existing reader cannot draw "`arbiter attempt <n>` unreadable" beside the readable attempts, and the line has no place in the frame.
`writeArbiterJSON` writes in place rather than by rename, so a lock-free read can catch a half-written file.

### [LOW] An attempt without a verdict reads running forever

**Location:** `_lyx/discussion/decision-record.md:63`
**Issue:** `ArbiterAttempt` records a `Deadline`.
An arbiter that died or timed out writes no verdict, and D3 would keep showing it as `running`.

### [LOW] D5 omits the pause record's retention bound

**Location:** `_lyx/discussion/decision-record.md:87`
**Issue:** `pauserecord.Update` drops resumed entries older than `Retention`.
A long-paused task's old visits then count their pause as row time again, the very symptom the task fixes.
D5 names retention only as a reassurance.

### [LOW] D8 states the guard bound in the wrong direction

**Location:** `_lyx/discussion/decision-record.md:120`
**Issue:** `configengine.CategoryGuard` is "a key that switches off or hollows a guard, a review or an approval gate".
The bound worth stating is that no local override or `run_settings` can switch the self-review forks off.
D8 states instead where they can be turned on.

## Rejected

- D10 versus the parity criterion (focus 5): not a defect.
  D10 line 137 states the premise that the notice sender has no advisor to report on, and the code bears it out: notices are queued only per advisor in `seatengine.Engine`, and the chair's `HoldDeferral` is `advisorWork.outstanding`.
  The Plan-Write amendment is carried by D10 itself (line 136), and `MultiLLMProducer.prepareFreshSpawn` today returns only an error, so the gap the record names is real.
- D8's claim that the chair stencil covers a lead with no advisors: verified in `contracts/stencils/loom/loom-template-discussion-chair.md` and `contracts/stencils/seat/seat-directive-chair.md`, both of which carry the no-advisor branch.
- D4 and D5 premises: `routed_to` exists and today is set only on reroute entries; `episodeStuckCount` reads it only on reroutes; `pauserecord.CoverageOf` covers hub entries and runs a live entry to now; `Excluded` counts overlaps once.
- Goal and Scope traceability (focus 7): each Scope In and Out item traces to a Decision or Constraint.
  A fuller list of the stencil and refusal-spec files the change touches would be call-site enumeration, which the rubric excludes.
