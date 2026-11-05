---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D9 steps 1-3, with D8's filing-time return
    summary: Raddle-File writes the raddle store and the part records before the board write that runs D8's re-check, and a retry skips any step whose result exists, so after a return to Raddle-Write the re-approved proposal is filed beside a stale stored proposal and stale part records
    fix: In D9, run D8's re-check before step 1 and again inside step 3, and make steps 1 and 2 key their skip on the approved proposal's hash, so a result written for an earlier proposal is rewritten, not skipped. A D8 return deletes the step 1 and step 2 outputs the failed attempt wrote. Extend the D9 retry criterion to cover a re-check failure at step 3 followed by a re-approved, changed proposal.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md Open risks line 3, with D12
    summary: The Codex cache-write risk leaves open a fact the repository already settles. A Codex rollout's token_count carries input_tokens, cached_input_tokens, output_tokens and reasoning_output_tokens and no cache-write field. Codex also has no UsageReader today, so D12's each engine fills from what it reads has nothing to fill for Codex.
    fix: Delete the Open risks line on Codex. Add a Constraints fact that a Codex rollout's token_count events carry input_tokens, cached_input_tokens, output_tokens and reasoning_output_tokens and no cache-write field (internal/shuttleengine/codexengine/rollout_test.go, docs/research/codex-review-bench-2026-10-25/report.md). State in D12 that the Codex engine gains a UsageReader over the rollout's last total_token_usage. Say that its cache-write count is zero, not unknown, and its input count is input_tokens less cached_input_tokens. Add a D12 criterion for a Codex session's reading.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D13 and the Constraints PATTERN-no-hard-coded-choice bullet
    summary: D13 adds a second per-model price table under the report key prices, although claudeengine already prices each message through the MessagePricer seam. tools/tokencount already prices run transcripts with it. A session total of four counts cannot apply the one-hour cache-write rate or Haiku's long-prompt card, which the engine prices per message.
    fix: In D12, have the usage writer also record each session's dollar cost, priced per message at session end through the engine's MessagePricer, reached via shuttleengines, with an unpriced model left unknown. In D13, read dollars from those records and drop the prices config key. Replace the Constraints PATTERN-no-hard-coded-choice bullet, which covers language, engine and branch, not prices, with a PATTERN-shuttle-provider-seam bullet saying prices come only from the engine's pricer. Update the D13 criterion to match.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D9 step 2, with D3
    summary: Step 2 makes the part records before step 3 creates their entries. boardengine.StoreDecisionRecord, the one path that stores a record and sets the discussed flag, refuses an absent entry and a done one. The record never says how a part record becomes the stored record that D3's Record-Write reads.
    fix: In D9, state that step 2 writes the part records into the raddle store or the run tree only. Step 3's one board write then stores each filed entry's decision file and sets its discussed flag together with creating the entry, under the board lock, through a new boardengine write that files an entry with its record.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D7 Bound and D9
    summary: The record never says when raddle_auto_approve is read. The key can be set on in the pair's local config layer mid-run, which configcli does not refuse to a task session, and the board's own run_settings notice names that route. Raddle-File would then file with no hash check, and it never re-runs D6's checks.
    fix: In D7, fix the mode once at Raddle-Write's first spawn, record it in the run's state, and have every later Raddle-Write session and Raddle-File read that recorded mode, never the key. Have Raddle-File run D6's checks on the proposal before filing in both modes. Add the recorded-mode rule to D7's Bound and a D7 criterion for it, under which a key changed mid-run does not change the mode.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D7 and D8 return path
    summary: An approval survives a D8 return. If the new session leaves proposal.yaml unchanged, the old hash still matches, so Raddle-File files against a moved board with no new approval, although D8 says a new approval is needed.
    fix: In D8, state that a return to Raddle-Write clears the recorded approval, so Raddle-File in interactive mode files only after a raddle-approve made after the return. Add that case to the D8 criterion.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D7, D9 step 1 and D14 in auto mode
    summary: Only raddle-propose stores the cold proposal, and it runs only with the key off. In auto mode, D9 step 1's cold proposal and D14's cold-against-approved comparison have no defined input.
    fix: In D7, state that in auto mode the first gate pass of the run stores the cold proposal, once per run, and the final gate-passing proposal is the approved one. Add a D7 criterion for auto mode storing the cold proposal.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D7 second bullet and D8 return
    summary: The waiting notice goes out only on the first passing raddle-propose of a run. After a D8 return in interactive mode, the new session needs a fresh approval and nobody is told.
    fix: In D7, post the waiting notice on the first passing raddle-propose of each Raddle-Write session, and keep the cold proposal stored once per run. Change the D7 criterion from first pass only to first pass per session.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D9 step 3 and Bound
    summary: Step 3 marks the run's own entry done while steps 4 and 5 still run. SetRunStatus skips a done entry, so a halt in the beam or notice step, or after the second return, never shows on the board.
    fix: In D9, state what a halt after step 3 shows and where. Either the halt notice to the parent orch names the halted step, since the board cannot, or the entry is marked done in a final board write after step 5. Add the chosen behaviour to D9's Bound.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 Bound
    summary: The bound that only a raddle run turns a Sketch into tasks does not hold. A note write can drop the sketch label, and the entry then promotes with any recipe while keeping its discussed flag and stored record, which D3 then seeds.
    fix: In D2, key the promote rule on the discussed flag with a stored record rather than on the sketch label, so any discussed note promotes only with recipe raddle. Alternatively, state in the Bound that removing the label is the operator's sanctioned way out, and what it lets through. Add the chosen case to the D2 criterion.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 first bullet and last line
    summary: Gone from the board is read as landed, but lym board remove deletes an entry that never landed. Raddle-File then drops a depends_on on an abandoned prerequisite and files the part released.
    fix: In D8, count an entry gone from the board as landed only when the board's history shows it done before prune. Otherwise fail the re-check, naming the entry. If that history cannot be read, state in D8's text that a removed entry's dependency is dropped as if landed, as what the relaxed guard lets through.
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6 first check, with D5 parts and D9 step 2
    summary: D6 requires every decision, criterion and risk to be carried by a part, but not every PATTERN change. A Sketch PATTERN change no part carries is dropped at filing, and one carried by two parts lands twice.
    fix: In D6's first check, add that every PATTERN change of the Sketch is carried by exactly one part. Add the case to the D5 and D6 criterion.
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D13, with D3 and D4
    summary: The cost fit is keyed by recipe and profile only. Most past tapestry runs paid for Discussion, and every raddle-filed part starts discussed, so the fit overstates each filed part's fixed cost and D14 reports that bias as the agent's error.
    fix: In D12, have Seed also record whether the run started from a stored record. In D13, key the fit by recipe, profile and that flag. Name the flag in the D12 and D13 criteria.
  - id: F14
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D12 and D13
    summary: D13 fits a fixed cost plus a cost per 100 changed lines, and D12 records the size only in Finalize, so a record-only run such as motif or raddle has no size, and neither decision says how it enters the fit.
    fix: In D13, state that a run with no Finalize size record reports its mean only, with no per-size fit, for its recipe and profile.
  - id: F15
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 second bullet
    summary: The re-check sees only tasks and projects created after the baseline. An existing entry outside board_checked that gains one of the Sketch's module labels later passes unseen.
    fix: In D8's second bullet, check every task or project outside board_checked whose revised stamp is later than the baseline and that carries a module label the Sketch carries, not only those created after it.
  - id: F16
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D9 retry rule
    summary: D8's free-slug check runs inside step 3's write, and the record names no marker for step 3 done. A retry after a crash that followed the write finds its own slugs taken and returns Stuck instead of skipping.
    fix: In D9, state that step 3 counts as done when the run's entry is already done and every filed slug exists, and that the retry checks that before D8's re-check.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D9 Bound and Constraints facts
    summary: D9 rests on the run lock admitting a done status and on status writes skipping a done entry, both true in code but uncited among the facts. Its claim that Land prunes the entry fails for an entry carrying notify_on_land, which Prune keeps and whose landing notice fires at step 3.
    fix: Add two Constraints facts. First, the run lock compares no Status (internal/boardengine/runlock.go). Second, Board.SetRunStatus and MakeSketch skip a done entry (internal/boardengine/board.go, internal/boardengine/sketch.go). In D9 step 3, clear the entry's notify_on_land in the same write.
  - id: F18
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D10 and D9 step 1
    summary: The raddle store is written beside board.json outside the board's write path. The record never says how it is committed and reaches origin. StoreDecisionRecord's own refusal notes that the run ledger sets aside a board commit whose entries have all ended.
    fix: In D10, state that the store is written inside step 3's one board write and commit, so it reaches origin with the filed entries. Add a D10 criterion that another machine's lym board raddle reads it.
  - id: F19
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md PATTERN changes, PATTERN-run-usage-record bullet
    summary: The new entry says cost reports read only the usage files. D13 takes active hours from reportengine's timing, per the Notes, and D14 reads the raddle store, so the entry text contradicts the decisions it pins.
    fix: Reword the PATTERN-run-usage-record bullet's last clause to say that a cost report reads token counts only from those files.
  - id: F20
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Open risks line 1
    summary: The first Open risks line carries a requirement on the stencil, that the agent say so in its cost lines when a cell reads too few runs. That is a decision, not a risk, and it sits outside D4.
    fix: Move the stencil rule into D4 as a sentence, and add it to the Review D4 criterion. Delete the Open risks line, or keep only its first clause as the risk.
---

### [BLOCKING] A filing-time return leaves stale store and part records that the retry skips

**Location:** `decision-record.md` D9 steps 1 to 3, with D8's filing-time return
**Issue:** D9 orders the steps as store, then part records, then the board write that runs D8's re-check, and it skips each step whose result exists.
When the re-check fails, D8 returns to Raddle-Write, and the operator approves a revised proposal.
The next Raddle-File then finds the step 1 and step 2 outputs already there and skips them.
The store keeps the old proposal, and the part records belong to the old split, possibly under old slugs.
D10's lookup by filed slug, D11's hand-off trigger and D14's comparison all read that stale store.
Filed parts whose slugs survive the revision build from stale records through D3.

### [MEDIUM] Codex token split left as a risk the repository answers

**Location:** Open risks line 3, D12
**Issue:** `internal/shuttleengine/codexengine/rollout_test.go:23` and `docs/research/codex-review-bench-2026-10-25/report.md:196` show the rollout's `token_count` fields: `input_tokens`, `cached_input_tokens`, `output_tokens`, `reasoning_output_tokens`, and no cache-write count.
`rollout.go` reads only `last_token_usage.total_tokens`, and Codex implements no `UsageReader`.
The stencil's Step 2 settles such a fact during Discussion. Leaving it as a risk hands it downstream, and the risk's claim that cost is understated is unverified.
Also flagged by the bound-breaker lens.

### [MEDIUM] A second price table beside the engine's pricer

**Location:** D13; Constraints, `PATTERN-no-hard-coded-choice` bullet
**Issue:** `internal/shuttleengine/claudeengine/pricing.go` prices each message through `shuttleengine.MessagePricer`, which reaches readers through `shuttleengines.RecordReaders()`.
`tools/tokencount` already uses it to price run transcripts by segment and recipe.
D13's config key `prices` duplicates that table outside the provider seam.
It also prices session totals, which lose the one-hour cache-write share (`CacheWriteOneHour`) and Haiku's per-message long-prompt card.
`PATTERN-no-hard-coded-choice` governs language, engine and branch, so it does not justify the key.
The Constraints fact that no lym code records a run's token cost is true, but the record never weighs the pricer or `tokencount`.

### [MEDIUM] Part records have no path into the board store

**Location:** D9 step 2, D3
**Issue:** `boardengine.StoreDecisionRecord` (`internal/boardengine/decision.go:43-50`) refuses an absent entry and a done one, and step 2 runs before step 3 creates the entries.
D3's Record-Write reads only a stored record, so the record must say how a part record reaches the store.

### [MEDIUM] Auto-approve key read time is unbounded

**Location:** D7 Bound, D9
**Issue:** `internal/boardcli/runsettings.go:77-79` tells a running pair to change settings through `lym config <module> --layer local --set`.
`internal/configcli/hubwide.go:49-61` refuses only hub-wide writes to task sessions, so a local-layer write from the pair goes through.
A key turned on after approval skips the hash match, and Raddle-File never re-runs D6.
The interactions lens and the handler found the same gap.

### [MEDIUM] An approval survives a return to Raddle-Write

**Location:** D7, D8
**Issue:** The hash binds the approval to the file, not to the board state it was given against.
An unchanged file after a return matches the old approval.

### [MEDIUM] No cold proposal in auto mode

**Location:** D7, D9 step 1, D14
**Issue:** Only `raddle-propose` defines the cold proposal, and it runs only with the key off.
The walkthrough lens found the same gap.

### [MEDIUM] No waiting notice after a return

**Location:** D7 second bullet, D8
**Issue:** The notice goes out once per run, and the D7 criterion pins that.
After a filing-time return, the new interactive session waits for an operator who was never told.
The interactions and bound-breaker lenses found the same gap.

### [MEDIUM] Marking the entry done mid-run hides later halts

**Location:** D9 step 3 and Bound
**Issue:** `Board.SetRunStatus` (`internal/boardengine/board.go:203`) skips a done entry.
Once step 3 runs, a halt in step 4 or 5 never shows on the board.

### [MEDIUM] D2's bound breaks when the label is dropped

**Location:** D2 Bound
**Issue:** `IsSketch` (`internal/boardengine/sketch.go:35-37`) reads only the kind and the label.
A relabelled discussed note promotes with any recipe, and D3 seeds it from the Sketch's record.

### [MEDIUM] Gone from the board is not landed

**Location:** D8
**Issue:** `Board.RemoveTask` (`internal/boardengine/board.go:224-229`) removes an entry whatever its status.
D8 then drops a dependency on work that never landed.

### [MEDIUM] PATTERN changes have no coverage check

**Location:** D6, D5, D9 step 2
**Issue:** Each part carries only its own PATTERN changes, and no check makes the parts together carry all of the Sketch's.

### [MEDIUM] The cost fit ignores whether a run started discussed

**Location:** D13, D3, D4
**Issue:** Raddle-filed tapestry parts start at Plan, but the fitted fixed cost includes Discussion from undiscussed runs.

### [LOW] Runs without a size record

**Location:** D12, D13
**Issue:** Record-only recipes have no Finalize, so they have no size to fit against.

### [LOW] The re-check misses relabelled entries

**Location:** D8 second bullet
**Issue:** Only entries created after the baseline are compared.

### [LOW] No done marker for step 3

**Location:** D9
**Issue:** A crash after the board write makes the retry's free-slug check fail on its own slugs.

### [LOW] Uncited facts and the notify_on_land exception

**Location:** D9 Bound, Constraints
**Issue:** The two facts D9 rests on are true in `runlock.go` (`scopeChanged`, lines 185-217) and in `board.go:203` / `sketch.go:77`, but neither is cited as a fact.
`Store.Prune` keeps a done entry carrying `notify_on_land` (`internal/boardengine/store.go:793`).
The bound-breaker lens found the same exception.

### [LOW] The raddle store's route to origin

**Location:** D10, D9 step 1
**Issue:** No decision says that the store's files are committed with a board write.

### [LOW] The new PATTERN entry's text overreaches

**Location:** PATTERN changes, `PATTERN-run-usage-record`
**Issue:** Cost reports also read timing records (Notes) and the raddle store (D14).

### [LOW] A stencil requirement filed as an Open risk

**Location:** Open risks line 1
**Issue:** Plan writes the stencil from D4, and the rule that the agent say "too few runs" in its cost lines sits outside it.

## Rejected

- The rubric's task-filing bullet does not apply to D9 or D2. Filing tasks on approval is the purpose the operator's board entry asks for, not a follow-up the run decides to file. Raddle-File's board write carries no strand name, since only `boardcli` sets `Config.StrandName` (`internal/boardcli/cli.go:159`, `internal/boardengine/board.go:155`). `checkTaskCreation` still refuses the raddle agent's own promote or upsert, so D9's Bound holds.
- That promotion releasing the entry is a new bypass: `Store.Promote` keeps a Sketch's nil status, so a promoted note already reads as released today (`internal/boardengine/store.go:759-785`).
- The relocation test over Constraints: each fact names its source and the decision resting on it, which the plan writer needs, so nothing moves. Open risks lines 2 and 4 are risks the plan writer acts on and stay. Line 1 is F20.
