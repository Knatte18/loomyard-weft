---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:44
    summary: D2 allows nested blocks (Q9) but its rewrite of empty on_done and its refusal of inlined edges leaving their recipe contradict each other under nesting, and a block row can never close a sub-recipe
    fix: "In D2, state the expansion order (innermost block first) and that the refusal of an inlined on_done, on_stuck or on_reroute naming a row outside its own recipe reads each row's source edges, so an edge D2 itself sets (a rewritten empty on_done, a nested block row's on_done set to its sub entry) is exempt. State which exit the rewrite uses: an inlined row's empty on_done becomes the exit of the innermost block enclosing it. In D1, either let a block row's on_done be empty, rewritten like any inlined row's to the enclosing block's exit (and refused empty outside every block), or add a Bound line saying a sub-recipe cannot end in a block. Add a nested-block row to the D2 acceptance bullet: a parent holding a block whose recipe holds a block expands with each inner row's path outer/inner and each empty on_done set to its innermost block's exit."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:66
    summary: Done names two events, the block row's Done recorded at entry (D3) and the block's Done at the sub-recipe's end (Goal, D1, D4); history shows the block done before its first inlined row runs, and D4's condition misses an exit reached by a skipped inlined row
    fix: "Make one reading hold across Goal, D1, D3 and D4. In D3, say that the block row's Done entry records entering the block, not the block's end, and that `lym loom status` shows it as entered (or name the history outcome that marks it so). In D4, say the block's end has no history entry of its own and is read as the first entry whose producer carries no block path (or the exit's entry), and widen its condition to an inlined row whose expanded on_done is the block's exit returning Done or being skipped by its entry gate. In the Goal and D1, replace 'whose Done is the sub-recipe's end' and 'once the block is Done' with wording that matches: the block ends when the run reaches its exit. Adjust the D3 acceptance bullet so it states what the block row records on entry."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:68
    summary: D3 makes the block gate bypassable by goto, onto the block row or onto any inlined row, including one inside a block the gate passed over, with no stated bound
    fix: "Add to D3's Bound line: only `lym loom goto` bypasses the block gate; goto moves only a halted run, and only back to a row CheckGotoTarget admits; a goto onto an inlined row of a passed-over block runs that row with the block gate never held, which is the goto caller's decision; a re-launch onto the block row clears the entered row and evaluates the gate again. Add to the D3 acceptance bullets: a re-launch onto the block row evaluates its gate."
  - id: F4
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:169
    summary: The board's scope makes the new recipe's name the operator's to approve; the record proposes arras and leaves the approval open (Open risks, Q8), which only the parent can settle
    fix: The parent obtains the operator's approval of the name arras, or a replacement name, and records it with lym loom decision add; the writer then drops the Open risks line and marks Q8 resolved.
  - id: F5
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:65
    summary: D3 states that a skipped row routes to its skip target but not that readers which reconstruct a skip's route from history do the same; gotoAdmitted reads a skipped entry's route as OnDone, which on a block row is the sub entry
    fix: "In D3, after 'a skipped row routes to it when it is set, and by OnDone otherwise', add: every reader that reconstructs where a skipped entry routed, the goto admission's routed row included, reads the skip target the same way. Extend the plan-writer note on internal/shedengine/entry.go to name the goto admission in internal/shedengine/goto.go as a second reader."
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:71
    summary: D3's bound says the skip target jumps past every inlined row, but nothing refuses a block row whose on_done names one of its own inlined rows or itself, so a pass-over could land inside the block or loop back onto the block row
    fix: "Add to D2's refusal list: a block row whose on_done names the block row itself or a row its own expansion inlines. Add that case to the D2 refusals acceptance bullet."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:114
    summary: D8's bound says profile_recipe changes no budget, but tapestry's profile blocks set the review budgets (discussion, plan and code max bounces and hard caps), which profile_recipe tapestry applies to an arras task
    fix: "Replace D8's Bound line with: profile_recipe chooses which recipe's profiles.yaml block and default profile, review budget keys included, a task's settings come from; it changes no row or config module, only a top-level key of an embedded recipe sets it, and Expand refuses it on a sub-recipe (D2)."
---

### [MEDIUM] Nested blocks: rewrite order and edge refusal contradict

**Location:** `_lym/discussion/decision-record.md:44` (D2), with D1 at `:32` and Q9 in `support-log.md:67`
**Issue:** Take a parent P holding a block on M, where M holds a block on I.
If I is expanded into M first, I's rewritten last `on_done` names a row of M, and M's inner block row's `on_done` names I's entry. Both are inlined edges naming a row outside their own recipe, so D2's refusal at `:53` refuses the nesting Q9 allows.
If P's rewrite runs first, the rows are not yet expanded and the exit the rewrite uses is undefined.
D1 also requires `on_done` on every block row, and D2 refuses one that leaves its recipe, so a sub-recipe can never end in a block.
Reported by bound-breaker, interactions and walkthrough.

### [MEDIUM] The block's Done means two events

**Location:** `_lym/discussion/decision-record.md:66` (D3), against `:5` (Goal), `:32` (D1) and `:77` (D4)
**Issue:** D3 records the block row Done when the gate holds, before any inlined row runs (the Block producer returns Done at once, and run.go routes it by `OnDone` to the sub entry).
The Goal, D1 and the board entry make the block's Done the sub-recipe's end, and D4 defines that end as an inlined row reaching the exit, an event no history entry records.
`lym loom status` therefore shows `Cartoon` done while Discussion-Write is still running.
D4's condition also misses an exit reached through a skip: an inlined row with an `entry_gate` routes its skip by `OnDone` as a Done does (`internal/shedengine/entry.go:50-70`).
Reported by walkthrough, interactions and bound-breaker.

### [MEDIUM] Goto bypass of the block gate has no bound

**Location:** `_lym/discussion/decision-record.md:68` (D3)
**Issue:** D3 states that a goto enters the block without evaluating its gate, but its one `Bound:` line covers only the skip target.
`gotoAdmitted` admits every row before the current one in list order (`internal/shedengine/goto.go:334-341`), and inlined rows sit before the block's exit, so after a pass-over a goto onto Discussion-Write is admitted and runs Discussion inside a block whose gate said to pass over it.
D3's "as a goto bypasses any target's gate" also omits that a re-launch onto the block row clears the entered row and evaluates the gate (`goto.go:243-244`).
The rubric requires a Decision that makes a guard bypassable to state what bounds the bypass.

### [MEDIUM] The recipe name awaits the operator

**Location:** `_lym/discussion/decision-record.md:169` (Open risks), `support-log.md:66` (Q8)
**Issue:** The board entry's scope says "Discussion proposes the new recipe's name, and the operator approves it."
The record proposes `arras` and leaves approval open.
No writer choice can settle it, so it goes to the parent.

### [LOW] A skipped entry's route is still reconstructed from OnDone

**Location:** `_lym/discussion/decision-record.md:65` (D3)
**Issue:** `gotoAdmitted` takes a skipped entry's routed row from `def.OnDone` (`internal/shedengine/goto.go:317-318`).
After D2, a block row's `OnDone` is the sub entry, so when `current_producer` names no row, a goto admits the block's first inlined row as routed and does not admit the exit the run went to.
D3 states the skip-target rule for the skip itself only, and the plan-writer note names only `entry.go`.

### [LOW] A block row's on_done can point into its own block

**Location:** `_lym/discussion/decision-record.md:71` (D3 bound), D2 refusals at `:49-54`
**Issue:** D2's edge refusal covers inlined rows only, and Parse checks no routing (`internal/shedbuild/parse.go:37-40`).
A block row whose `on_done` names one of the rows it inlines passes every listed check and `validate` (the name exists after expansion), so a pass-over lands inside the block, against D3's bound.
A block row whose `on_done` names itself yields a skip target equal to its own name: `validate`'s self-reference check reads `OnDone`, which now holds the sub entry, so only the entry gate's chain limit (`entry.go:33-36`) stops the loop, and it fails the run.

### [LOW] D8's bound denies the budgets it carries

**Location:** `_lym/discussion/decision-record.md:114` (D8)
**Issue:** The shipped profiles' tapestry blocks set `discussion_max_bounces`, `plan_max_bounces`, `code_max_bounces` and their hard caps (`internal/runprofile/template.yaml:18`, `:27`, `:37`).
`profile_recipe: tapestry` applies those to an `arras` task, so "it changes no row, budget or config module" is false for budgets.

## Rejected

- bound-breaker F6, `cartoon`'s `terminals` value unstated: Parse requires a non-empty list and D2 says a sub-recipe's terminals are never read, so any value serves; choosing it is plan detail.
- interactions F3, a block cannot end a sub-recipe: merged into F1.
- interactions F4 and bound-breaker F4: merged into F3 (goto into a passed-over block) and F5 (the skipped-route reader).
- bound-breaker F5, D4's condition misses a skipped last row: merged into F2.
- Peer run `decision-record-on-board` adding `approve_seam: discussion` to the Discussion-Bouncer row: no design conflict. Both records name the same coordination, and D9's whole-config pin against tapestry's live row fails whichever task lands second until `cartoon` carries the key, in either landing order. The Open risks line is accurate.
- Deferred research: none found. The record settles how a skip routes today (the plan-writer note on `entry.go`) and how profiles key on recipe name (`internal/battencli/runsettings.go` is the one reader, and it keys on `settings.Recipe`).
- Board rules: no Decision edits this task's board entry, and none files or promotes a task. The Scope Out follow-ups and Q10 are named, not filed.
- Leanness: `decision-record.md` holds no deliberation narrative that belongs in the support log, and the support log's review bullets carry no requirement missing from the record.
