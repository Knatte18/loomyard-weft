---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, Constraints (JSON output cap bullet), D1, D6, Acceptance criteria (8 KiB budget bullet)
    summary: The claim always hands out the first message whatever its size and a body may be 64 KiB, so a post over 10,000 characters reaches the session as a preview while creel journals it read; the Constraint that the 8 KiB budget keeps each field under the cap is false.
    fix: >-
      In decision-record.md, add to D1 (or a new decision) how a single message whose line exceeds what fits in one hook field is delivered,
      with the evidence it rests on (internal/creelengine/verbs.go ClaimAtBoundary always hands out the first message; internal/creelengine/pool.go maxBodyBytes = 64 << 10).
      Pick one: lower maxBodyBytes to the 8 KiB boundary budget so post refuses a longer body with its existing write-it-to-a-file way forward,
      or have the claim deliver an oversized message as a fixed line naming its id, sender and size and the command that reads it in full, leaving it unread until that read.
      Rewrite the Constraints bullet on the 10,000-character cap so it states the bound the chosen rule gives, and add an acceptance criterion for one message larger than the budget.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, D2 (Creel counts paragraph), D4 ring condition, D5 at-cap trigger, Acceptance criteria (cap bullet)
    summary: A woken turn's Stop carries stop_hook_active true and the reset reaches the daemon only while the flag stands, so an idle-only receiver's count climbs across unrelated wakes to the cap; and D2 does not order the reset against the cap check, so the turn a cap-triggered bell starts can claim nothing and strand the mail.
    fix: >-
      In D2, state that a Stop with stop_hook_active false resets the count before the cap check, so the Stop of a turn a D5 bell started claims the mail.
      Research in the Claude Code 2.1.312 bundle whether an asyncRewake wake resets Claude Code's own consecutive-continuation count, record the answer with its evidence in Constraints,
      and set creel's reset rule from it so that successive idle wakes with no mid-turn claim between them do not accumulate toward the cap (for example, a ring starts a fresh count at 1 when the wake resets Claude Code's count).
      Add acceptance criteria: three successive idle wakes of one strand each ring and deliver through the Stop block; at the cap, D5's bell is typed and the bell turn's Stop delivers the mail.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, D2 Bound, D4 once-per-set wake and Bound, D5 Bound
    summary: The creel daemon restarts on every binary replacement, and the record does not say the continuation count and woken-set marks survive it, so a restart mid-chain lets creel block past Claude Code's cap of 8 and lose journaled-read mail, and re-rings a set already woken.
    fix: >-
      In D2, state what a restarted daemon holds for each strand's count: either it journals the count, or it starts every strand's count at the cap so nothing blocks or rings until the strand's next reset.
      In D4's Bound, add that a restarted daemon may ring a set once more, as D5's Bound says of the bell.
      Add an acceptance criterion that a daemon restart mid-chain never lets the chain's blocks exceed creel's cap.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, D4 (idle check sentence), D5 (when the strand is idle, D4's reading)
    summary: The Delivery seam has no no-running-turn reading (a running turn and an active operator both answer not-idle with no stall, a draft answers draft), so D4's ring condition cannot be computed from it, and D5 typing under D4's draft-ignoring reading would type the bell into the operator's draft.
    fix: >-
      In D4, name the reading the ring uses as a change to the Delivery seam's Idleness answer (for example a separate running-turn report), and state which stalls (draft, too-short, no-input-box) hold the ring and which do not.
      In D5, replace D4's reading with the guarded send's own Delivery.Idle, so a draft, a too-short pane or a missing input box holds the typed bell as it holds every typed line today.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D5, D4 bell text, Open risks (third line)
    summary: A session launched before the deploy counts as shuttle-launched, so D5 bells it, but its Stop hook only records; the bell tells it to end the turn and not run lym creel read, so the mail stays unread, against the entry's once-it-is-restarted rule.
    fix: >-
      In D5, limit the bell to a strand whose session runs the new Stop delivery (for example a marker shuttle records in the strand's reed state at launch from this deploy on), and state that a strand without it gets no bell, as a hand-added strand gets none.
      In Open risks, drop the clause that D5 may bell pre-deploy sessions.
  - id: F6
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D1 (acknowledge-after-write stays), D2 (merged command prints the captured block)
    summary: In D2's merged command hook-read writes into the shell's capture, so creel acknowledges the claim before Claude Code reads it; a failed re-print or a hook killed by its timeout loses journaled-read mail, and D1 says the order stays as it is.
    fix: >-
      In D2, either have the merged command let hook-read write the block to the hook's standard output directly and decide on recording from hook-read's exit status, so the acknowledgement follows the write to Claude Code,
      or state the window in D2's Bound: mail claimed in a Stop block is lost if the command dies between the capture and its print.
  - id: F7
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, D2 (appends no Stop record)
    summary: The recording command writes a stamp line before the payload, and D2 does not say a blocked Stop skips the stamp too, so every blocked stop could leave an orphan stamp that ParseSessionSignals pairs with the next payload.
    fix: >-
      In D2, state that a blocked Stop appends neither the stamp line nor the payload, and that the stamp is written only together with the record, after the creel step decides.
  - id: F8
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, D7
    summary: post --interrupt to a hand-added strand stops its turn though D5 bells no such strand and it has no hooks, so nothing reaches it; and a store that fails after the key was sent leaves an interrupted turn with no post and no stated answer.
    fix: >-
      In D7, refuse --interrupt, before any key, for a receiver D5 excludes, naming the way forward (post without --interrupt and have the receiver run lym creel read, or interrupt it with the interrupt verb).
      State the answer when the store fails after the key: an error saying the turn was interrupted and the post was not stored.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D6 Rationale and Bound
    summary: D6's rationale that every hub session is launched by shuttle with the hooks is false by D5's own hand-added strands, which after D6 get agent mail with no notice, reminder or bell, and D6's Bound names no such case.
    fix: >-
      In D6, correct the rationale to every shuttle-launched session, and add to the Bound that a hand-added strand gets no signal of its mail and reads it only with lym creel read.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, D9 (after the deny notice when there is one)
    summary: The deny notice already rides one --append-system-prompt argument, and D9 does not say whether the sentence joins that argument or rides a second flag, which may keep only the last value.
    fix: >-
      In D9, state that the sentence is appended to the same single --append-system-prompt argument, after the deny notice and a separator, so one flag carries both.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, Constraints (internal/shell bullet), D9
    summary: The constraint stops at the pwsh Quote single-quoting the sentence; whether an embedded double quote reaches claude's native argument intact on Windows, where PowerShell versions and .cmd shims strip or re-split it, is left unresearched.
    fix: >-
      Research how the pwsh launch line passes an argument with embedded double quotes to claude on Windows (the PowerShell native-argument mode lym runs under and how claude is resolved), record the answer with its evidence in the internal/shell Constraints bullet,
      and, if the quotes do not survive, state in D9 how the sentence is carried there.
  - id: F12
    severity: LOW
    class: decision
    origin: lens:board-fit
    location: _lym/discussion/decision-record.md, D6, Constraints (future engine bullet), Notes for the plan writer
    summary: D6 leaves no channel for any creel post, program notices included, to reach a non-Claude session, which makes codex-engine's review-only first step waits on no creel work false and removes the typed cross-engine path creel-agents relies on; parent, accept this, or should those notes be updated when this lands?
    fix: >-
      Raise under Open risks, for the parent: D6 removes every pane path for posts, so a Codex session, such as a Codex reviewer, gets no creel post, the LOOM decision notice included, until codex-engine adds hooks, and creel-agents' typed cross-engine message goes; accept, or have the operator update the codex-engine and creel-agents notes.
---

### [BLOCKING] An oversized first message overflows the hook field and is lost

**Location:** `_lym/discussion/decision-record.md`, Constraints (JSON output cap), D1, D6, Acceptance criteria.
**Issue:** `ClaimAtBoundary` hands out the first message whatever its size (`internal/creelengine/verbs.go:162`, loop at 183–198), and a body may be up to 64 KiB (`internal/creelengine/pool.go:17`).
A 20 KiB post becomes one `reason` and one `systemMessage` over the 10,000-character cap the record cites, so Claude Code shows a preview, while `hook-read` acknowledges and creel journals the message read.
Today a program notice over 4 KiB falls back to the reminder and agent mail is only reminded; D6 removes both, so the whole 8–64 KiB range goes through the hook.
The Constraint "D1's 8 KiB claim budget keeps each field under the cap" is a false premise, and the Goal's "in full" fails silently.
All five code-reading lenses and the handler found this independently.

### [MEDIUM] The continuation count climbs across unrelated wakes, and the reset is not ordered against the cap

**Location:** `_lym/discussion/decision-record.md`, D2, D4, D5, Acceptance criteria.
**Issue:** The support log's own fact is that a rewake turn's Stop carries `stop_hook_active` true, and `hook-read`, which carries `stop_hook_active` to the daemon, runs only while the flag stands.
An orch that receives mail only while idle therefore gains a ring and a block per post and never resets: after three idle posts the cap stops rings and D5's typed bell becomes its usual path.
D2 also leaves open whether a `stop_hook_active`-false Stop resets before or after the cap check; if after, the turn a cap-triggered bell starts reaches its Stop at 6, claims nothing, and D5 never repeats the bell for the unchanged set, so the mail stays unread despite the bell's promise.
No acceptance criterion covers the cap-triggered bell or the delivery after it.
Whether a wake resets Claude Code's own count is the fact the reset rule should rest on, and the record does not give it.

### [MEDIUM] A daemon restart loses the count and the woken-set marks

**Location:** `_lym/discussion/decision-record.md`, D2 Bound, D4, D5 Bound.
**Issue:** The creel daemon exits on every binary replacement (`internal/creelengine/server.go:47`, `doc.go:36`).
With the count in memory, a strand five blocks into a chain restarts at 0, creel allows six more, and Claude Code overrides its eighth consecutive block whose mail creel already journaled read, the loss D2's Bound says cannot happen.
D4's waiters reconnect by design, so the restarted daemon also re-rings sets it already woke, which D4's Bound does not name.

### [MEDIUM] D4's idle reading does not exist in the seam, and D5 would type into a draft

**Location:** `_lym/discussion/decision-record.md`, D4, D5.
**Issue:** `Idleness` answers a running turn and an active operator alike as not idle with no stall, and a draft as `StallDraft` (`internal/creelengine/seam.go`; `internal/creelcli/delivery.go:342–347`), so "reports no running turn, where a draft does not count as busy" cannot be read from the seam, and the record names no new answer.
D5 types the bell "when the strand is idle (D4's reading)" and also "under every rule a typed line has today"; the two disagree on a draft, and the first would type the bell into the operator's draft.

### [MEDIUM] The bell misdirects sessions launched before the deploy

**Location:** `_lym/discussion/decision-record.md`, D5, D4 bell text, Open risks.
**Issue:** Shuttle sets `Provider` on every strand it launches (`internal/reedengine/state.go:41-43`), so a pre-deploy session counts as shuttle-launched and gets the bell, but its Stop hook only records (`internal/shuttleengine/claudeengine/settings.go:314`).
The bell tells it to end the turn and not to run `lym creel read`; ending the turn delivers nothing and the bell is never repeated, so the mail waits for a tool call in some later turn.
The board entry says such a session gets posts once restarted, and the record's own rejected alternative refuses the bell to hand-added strands on the same ground.

### [LOW] The merged Stop command acknowledges before Claude Code reads the block

**Location:** `_lym/discussion/decision-record.md`, D1, D2.
**Issue:** To decide whether to record, the merged command captures `hook-read`'s output and re-prints it, so `hook-read`'s write, after which the client acknowledges (`internal/creelcli/hookread.go`), lands in the shell rather than in Claude Code.
A failure between the capture and the print loses journaled-read mail, while D1 says the acknowledge-after-write order stays.

### [LOW] D2 is silent on the stamp line of a blocked Stop

**Location:** `_lym/discussion/decision-record.md`, D2.
**Issue:** `recordingCommand` writes a stamp line before the payload (`internal/shuttleengine/claudeengine/settings.go:286-289`), and `ParseSessionSignals` pairs payloads with the nearest earlier untaken stamp.
"Appends no Stop record" does not say the stamp is skipped too.

### [LOW] `post --interrupt` stops a hand-added strand it cannot deliver to

**Location:** `_lym/discussion/decision-record.md`, D7.
**Issue:** A hand-added strand gets no bell and has no hooks, so `post --interrupt` stops its turn and delivers nothing.
D7 also gives no answer for a store that fails after the key was sent.

### [LOW] D6's rationale is false for hand-added strands

**Location:** `_lym/discussion/decision-record.md`, D6.
**Issue:** "Every hub session is launched by shuttle with the hooks" contradicts D5's hand-added strands, which after D6 get mail with no signal at all; D6's Bound does not name them.

### [LOW] D9 does not say how the sentence shares the deny notice's flag

**Location:** `_lym/discussion/decision-record.md`, D9.
**Issue:** The deny notice rides one `--append-system-prompt` argument (`internal/shuttleengine/claudeengine/command.go:254, 357`); D9 says "after the deny notice" without saying whether that is the same argument or a second flag.

### [LOW] Windows delivery of the D9 sentence's quotes is unresearched

**Location:** `_lym/discussion/decision-record.md`, Constraints (`internal/shell`), D9.
**Issue:** The constraint shows that both `Quote` forms keep `"` in the shell string, but stops one hop short of the native argument `claude` receives under pwsh, where embedded double quotes may be stripped or re-split depending on the PowerShell version and how `claude` resolves.
The sentence's verbatim arrival on Windows is a fact the design rests on, left unestablished.

### [LOW] Non-Claude sessions lose every creel post

**Location:** `_lym/discussion/decision-record.md`, D6, Constraints, Notes for the plan writer.
**Issue:** The `codex-engine` note says its review-only first step waits on no creel work, and `creel-agents` relies on creel typing a cross-engine post in full; D6 removes every pane path for posts, program notices such as `LOOM`'s decision notice included.
The record mentions this only in a Constraint and a plan-writer note, where the parent does not see it as a question; whether to accept it or have the notes updated is the parent's call.

## Merged duplicates

No lens finding was rejected as a false positive; these were folded into a kept finding.

- goal-scope F1 (pre-deploy bell, class `scope`): kept, merged into F5 with class `design`, since the defect is the bell rule, not a missed site.
- interactions F5 (acceptance criteria miss the cap-triggered bell): merged into F2, whose fix adds the criterion.
- existing-paths finding 1 and walkthrough F1, interactions F3 (oversized message): merged into F1; existing-paths rated it MEDIUM, but the loss is silent and the premise false, so BLOCKING holds.
- bound-breaker F3 and walkthrough F2 (count never resets): merged into F2.
- bound-breaker F5 (no running-turn reading): merged into F4.
- interactions F4 and bound-breaker F7 (ack before delivery): merged into F6.
- walkthrough F4 and bound-breaker F8 (D7 hand-added strand and store failure): merged into F8.
