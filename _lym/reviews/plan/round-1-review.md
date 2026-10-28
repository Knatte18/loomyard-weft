---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    origin: lens:guard-reach
    location: _lym/plan/08-creel-delivery-seam-fields.md (Intent, PaneMail and its Bound); _lym/plan/09-creel-pane-path-by-engine.md
    summary: The added decision keeps the pane path for a Codex session, but the plan gives a Codex session no delivery at all, because a Codex session today exists only as a hand-added strand with no provider tag and PaneMail needs a provider.
    fix: Settle how a strand with no provider tag is treated, through the parent if the fixer may not choose. The option that carries the added decision is that card 8 sets PaneMail true when the persisted Provider is empty or names an engine that shuttleengines.ToolBoundaryRenderer does not find, so a hand-added strand (a Codex session included) keeps today's notice and reminder. Then rewrite card 8's Bound sentence and its TestHubDelivery row, which wants a no-provider strand as neither, to want it as PaneMail. Rewrite card 9's TestSmokeCreelClaudeStrand change so it no longer asserts that a hand-added strand's post is never typed, and rewrite the overview's second Shared Decision to match. If the parent rules instead that hand-added strands get nothing, record that ruling in the decision record as an Added after Discussion entry, since it narrows the added decision's Codex case.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/17-creel-hook-docs.md (Edit list); internal/orchengine/doc.go:386,406,423; internal/orchengine/pause.go:235
    summary: Card 17 rewrites the docs that describe the reminder flow but misses internal/orchengine, whose package doc and queueResumeLine comment still say the orch's mail and resume line are reminded.
    fix: In card 17, add `internal/orchengine/doc.go` and `internal/orchengine#Watcher.queueResumeLine` to the Edit list, after `internal/creelengine/doc.go`. Add an Intent sentence saying that the orchengine package doc's passages on mail kept unreminded during a cycle, old mail reminded after a failed withdraw, and the resume line reminded once the pause lifts, and queueResumeLine's doc comment, are rewritten to say the orch's mail arrives through its hooks (or the bell when idle) once no pause or hold mutes it. This target change passed lym loom validate-plan --overlay together with the target changes of F7 and F8.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:way-forward
    location: _lym/plan/15-creel-post-interrupt.md (Intent, the interruptTarget step and the store-failure test row); internal/creelengine/verbs.go:567-575
    summary: interruptTarget returns sent=true with an error when its own interrupted journal append fails after the key, which card 15 treats as a refusal with nothing stored, and the planned store-fails-after-the-key row lands on exactly that path rather than on post's store step.
    fix: In card 15's Intent, after the sentence on interruptTarget's refusals, add that an error interruptTarget returns together with true (the key was sent and only its interrupted record failed) is answered as a store failure after the key, with the same error card 15 already names for that failure (the turn was interrupted, the post was not stored, and the way forward is to post again without --interrupt). Split the TestStore_Mail row into two rows. In the first, the interrupt function closes the journal, so interruptTarget's append fails, and the row asserts that error. In the second, the store fails only after interruptTarget returns, through an interrupt function that leaves the journal writable and a fault on the post's own append, such as a Strands error scripted for the re-read, and the row asserts the same error.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/07-shuttle-idle-no-turn.md (Tests); internal/shuttleengine/session.go:215-277
    summary: No row of card 7 reaches the pane-probe NoTurn branch for an engine with InterruptGuard, which is Claude's production path when the hook reading is unknown, since every pane-probe row uses readinessEngine, which implements no InterruptGuard.
    fix: In card 7's Tests paragraph, add rows on an engine that implements InterruptGuard, fakeEngine (whose RunningTurn reads RUNNING in the capture), under an unknown reading. A pane with an input box holding a draft and no running turn reads NoTurn true with Idle false and a zero TurnEndedAt. The same pane showing a running turn reads NoTurn false.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/18-creel-hooks-llm-smoke.md (Intent, the interrupt and bell scenario)
    summary: The interrupt-and-bell scenario asserts the bell is typed exactly once after post --interrupt, which holds only when no waiter started by an earlier Stop is still live, and D5 says whether Claude Code keeps such a waiter alive across a later turn is undocumented, so the scenario rests on a fact the plan leaves to be found at run time.
    fix: In card 18's interrupt-and-bell scenario, state that the long turn interrupted is the session's first turn, the one started by its launch prompt, so no Stop has run and no waiter exists. Then the bell is the only wake, and the bell assertions stand. Replace the sentence that says a user interrupt leaves no waiter with that premise.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lym/plan/15-creel-post-interrupt.md (Intent, the wrapping of interruptTarget's refusals); internal/creelengine/verbs.go:583-585
    summary: Card 15 wraps interruptTarget's hold and pause-lifted refusals with post's own way forward but leaves the unreadable-pause refusal as it is, so a post --interrupt that fails there says nothing was interrupted, not that nothing was posted, and names no next step.
    fix: In card 15's Intent, add the unreadable-pause refusal to the wrapped ones. Its text is the refusal followed by "; nothing was interrupted or posted", then a way forward naming a post without --interrupt, which stores the mail for the receiver now, or post --interrupt again once the pause record reads. Add a TestStore_Mail row in which SetPausedError makes ClaimPause fail, and assert that no key was sent, nothing was stored and the text names the post without --interrupt.
  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/07-shuttle-idle-no-turn.md (Extends); _lym/plan/10-creel-hook-waiter-ring.md (Extends)
    summary: Cards 7 and 10 list under Extends a ref whose existing text their Intent rewrites. Card 7 changes the sentence on what SessionIdle answers (internal/shuttleengine/doc.go:446). Card 10 inserts hook-wait into the existing list of creel's waiting verbs (pattern/PATTERN-daemon-wakeups.md:27).
    fix: In card 7, move `internal/shuttleengine/doc.go` from Extends to the end of the Edit list and drop the empty Extends section. In card 10, move `pattern/PATTERN-daemon-wakeups.md` from Extends to the end of the Edit list, after `internal/creelengine_test#TestDaemon_Lifecycle`, and drop the empty Extends section. Both moves passed lym loom validate-plan --overlay together with F2's and F8's target changes.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/13-creel-hook-verbs.md (Edit, internal/creelcli#hookRead)
    summary: Card 13's Intent makes hookRead report whether it wrote the block beside its error, and hookReadExit needs the parsed event, so hookRead's results change, yet its Edit glyph carries no re-sign arrow.
    fix: "In card 13's Edit list, replace `internal/creelcli#hookRead` with `internal/creelcli#hookRead` -> `func hookRead(cmd *cobra.Command, engine string) (event shuttleengine.HookEvent, wrote bool, err error)`, and say in the Intent that hookCmd's RunE passes those three results to hookReadExit. This passed lym loom validate-plan --overlay together with F2's and F7's target changes."
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/13-creel-hook-verbs.md (TestHookReadExit)
    summary: Most of TestHookReadExit's rows pin exits that card 13's TestCreelCLI rows already reach through the real verb. Those are a Stop with a block written exiting 0, a Stop with nothing claimed exiting 1, a tool-boundary call exiting 0, and a refusal exiting 1 with its error on stderr. Only the case of a block written and then a failed acknowledgement is out of the CLI's reach.
    fix: In card 13, cut TestHookReadExit to the rows no TestCreelCLI row reaches. Those are a Stop with a block written and an acknowledgement error, which exits 0 and prints the error. Keep a Stop with nothing written and an error, which exits 1 and prints it, only if no TestCreelCLI row reaches it. Name TestCreelCLI as the cover for the rest.
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:way-forward
    location: _lym/plan/15-creel-post-interrupt.md (Intent, the CreelHooks refusal text); internal/creelengine/reminder.go:27-30
    summary: The no-hooks refusal says a pane-mail receiver gets a post by its typed notice, which holds only for program or EXTERNAL mail of at most 4 KiB, while agent mail gets a reminder and needs lym creel read.
    fix: In card 15's CreelHooks refusal text, replace "by its typed notice when its engine has no hooks" with "by its typed notice or reminder when its engine has no hooks".
  - id: F11
    severity: NIT
    class: consistency
    origin: lens:way-forward
    location: _lym/plan/10-creel-hook-waiter-ring.md (Intent, the HookWait refusal); internal/creelengine/verbs.go:131-133
    summary: HookWait's refusal of a non-strand sender names lym creel read as the way forward, which an EXTERNAL caller cannot take, since Read refuses EXTERNAL.
    fix: "In card 10, change the HookWait refusal to \"creel: only a strand's idle waiter hook waits for a wake, and <name> is no strand\", followed for a program sender by \"; read with lym creel read\" and for EXTERNAL by \"; wait for an answer with lym creel post --wait-reply\"."
  - id: F12
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/12-shuttle-hook-capability.md; internal/shuttleengine/claudeengine/capabilities.go:2
    summary: The capabilities.go file header still lists tool-boundary context rendering among Claude's capabilities after card 12 removes ToolBoundaryContext.
    fix: In card 12's Intent, after the sentence that deletes Claude.ToolBoundaryContext, add that capabilities.go's file header names creel's hook delivery (the hook-input parse, the visible block and the wake output) in place of tool-boundary context rendering. Change no target, since a file target beside the card's members of that file fails redundant-file-target.
  - id: F13
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/16-claude-trust-sentence.md; internal/shuttleengine/claudeengine/prepare_test.go:3
    summary: The prepare_test.go file header still says the file covers the --append-system-prompt notice after card 16 replaces that flag with --append-system-prompt-file.
    fix: In card 16's Tests paragraph, add that prepare_test.go's file header names the prompt file carrying the notice and the trust sentence in place of the --append-system-prompt notice. Change no target, since a file target beside the card's test members of that file fails redundant-file-target.
---

### [BLOCKING] A Codex session gets no delivery, against the added decision that keeps its pane path

**Location:** `_lym/plan/08-creel-delivery-seam-fields.md`, `_lym/plan/09-creel-pane-path-by-engine.md`

**Issue:** The added decision "Posts still reach a session without creel hooks" says a strand whose engine has no creel hook delivery, "such as a Codex session, keeps today's pane path".
Card 8 sets `PaneMail` only when the persisted `Provider` is non-empty and `shuttleengines.ToolBoundaryRenderer` finds no renderer.
Only shuttle sets a provider (`internal/shuttleengine/run.go:596`, `open.go:85`), and shuttle launches only registered engines, which today is only `claude`.
So a Codex session can exist only as a strand added with `lym reed add`, which carries no provider (`internal/creelcli/smoke_helpers_test.go:87-102`), and card 8's own Bound sentence says it then gets "no pane path either".
The `PaneMail` branch the decision requires is reachable only in tests, while the one case the decision names gets no notice, reminder, ring or bell, where it gets the pane path today.
How an unknown-engine strand is treated is a choice the decision record does not settle explicitly: D6's Bound gives an unmarked strand nothing, and the added decision narrows D6.
That makes this a `decision` finding.

### [MEDIUM] Card 17 misses orchengine's reminder passages

**Location:** `_lym/plan/17-creel-hook-docs.md`; `internal/orchengine/doc.go:386,406,423`; `internal/orchengine/pause.go:235`

**Issue:** The decision record's Scope covers "the docs and stencils that describe the old reminder flow", and D11 has the docs follow the new delivery.
The orch strand is a marked Claude strand, so after this task its mail and its pause resume line arrive through its hooks, never a reminder.
The orchengine package doc still says mail is "unreminded" during a cycle, that old mail "is reminded", and that the resume line is mail "which creel reminds once the pause is lifted".
`Watcher.queueResumeLine`'s doc comment says "Creel reminds mail once the pause is lifted".
No card edits either.

### [MEDIUM] post --interrupt hides a stopped turn when interruptTarget's own record fails

**Location:** `_lym/plan/15-creel-post-interrupt.md`; `internal/creelengine/verbs.go:567-575`

**Issue:** D7 requires a store that fails after the key to answer that the turn was interrupted and the post was not stored, naming the repost.
`interruptTarget` sends the key and then appends `eventInterrupted`, returning `true` with the append's error when that fails.
Card 15 treats every `interruptTarget` error as a refusal with nothing stored.
So this after-key failure answers a bare journal error.
The card's own test row, an interrupt function that closes the journal, makes exactly this append fail, so it never reaches the post's store step that the row means to pin.
The guard-reach and way-forward lenses each reported this defect; the two reports are merged here.

### [MEDIUM] Card 7 leaves Claude's pane-probe NoTurn branch untested

**Location:** `_lym/plan/07-shuttle-idle-no-turn.md`

**Issue:** For an engine with `InterruptGuard`, card 7 sets `NoTurn` from `readBox` and `RunningTurn` when the hook reading is unknown.
Claude implements `InterruptGuard`, so this is the production path that decides D4's ring for a marked Claude strand whose reading is unknown.
Every pane-probe row of the card runs on `readinessEngine`, which by the card's own text implements no `InterruptGuard`, so no row reaches the branch.

### [MEDIUM] The LLM interrupt-and-bell scenario rests on undocumented waiter lifetime

**Location:** `_lym/plan/18-creel-hooks-llm-smoke.md`

**Issue:** The scenario asserts that after `post --interrupt` the pane shows the bell typed exactly once, on the premise that "no waiter reaches the stopped session".
D7 says the stopped session is woken through D5's bell, "or D4 when a waiter is still live".
D5's rationale says whether Claude Code keeps an older waiter alive across a later turn is undocumented.
If the interrupted turn followed an earlier Stop, that Stop's waiter may still be registered, card 10 rings it, and no bell is typed.
The scenario's verdict then depends on a fact the plan leaves to be found at run time.
The guard-reach lens reported this independently, at LOW.
It is kept at MEDIUM because the scenario can fail with no defect in the code.

### [LOW] The unreadable-pause refusal of post --interrupt is not wrapped

**Location:** `_lym/plan/15-creel-post-interrupt.md`; `internal/creelengine/verbs.go:583-585`

**Issue:** Card 15 gives the hold refusal and the pause-lifted refusal post's own text ("nothing was interrupted or posted") and post's way forward.
`unreadablePauseRefusal` passes through unchanged.
It says only "so nothing was interrupted: <err>", does not say the post was not stored, and names no next step.

### [LOW] Extends on refs whose existing text the card rewrites

**Location:** `_lym/plan/07-shuttle-idle-no-turn.md`; `_lym/plan/10-creel-hook-waiter-ring.md`

**Issue:** Card 7's Intent says "The doc's sentence on what `SessionIdle` answers gains the two fields".
That rewrites the existing sentence at `internal/shuttleengine/doc.go:446`.
Card 10's Intent puts `hook-wait` into the existing list "(`post --wait-reply`, `inject --wait` and `read --wait`)" at `pattern/PATTERN-daemon-wakeups.md:27`, which rewrites that bullet.
Extends is only for adding without changing what is already there.

### [LOW] hookRead is re-signed with no arrow

**Location:** `_lym/plan/13-creel-hook-verbs.md`

**Issue:** Today `hookRead(cmd *cobra.Command, engine string) error` (`internal/creelcli/hookread.go:55`).
Card 13 has it report whether the block was written beside its error, and `hookReadExit(event, wrote, err)` needs the event that hookRead parses.
So its results change, but the Edit glyph has no re-sign arrow.

### [LOW] TestHookReadExit repeats exits TestCreelCLI already pins

**Location:** `_lym/plan/13-creel-hook-verbs.md`

**Issue:** Card 13's TestCreelCLI rows already pin these exits through the real verb:
- a Stop that writes a block exits 0;
- a Stop with no mail exits 1, printing nothing;
- a tool-boundary call exits 0;
- a refused input exits 1, with the refusal on stderr.

TestHookReadExit tests the private decision helper over the same cases.
Only a block written and then a failed acknowledgement is out of the CLI's reach, as the card says.

### [NIT] The no-hooks refusal promises a typed notice that agent mail never gets

**Location:** `_lym/plan/15-creel-post-interrupt.md`; `internal/creelengine/reminder.go:27-30`

**Issue:** `isNotice` carries only program or EXTERNAL mail of at most 4 KiB as a typed notice.
Agent mail to a pane-mail strand gets a reminder.

### [NIT] HookWait's refusal names a read EXTERNAL cannot do

**Location:** `_lym/plan/10-creel-hook-waiter-ring.md`; `internal/creelengine/verbs.go:131-133`

**Issue:** `Read` refuses EXTERNAL with `errExternalHoldsNothing`, so "read with lym creel read" is no way forward for an EXTERNAL caller.

### [NIT] The capabilities.go header goes stale

**Location:** `_lym/plan/12-shuttle-hook-capability.md`; `internal/shuttleengine/claudeengine/capabilities.go:2`

**Issue:** The header says the file holds "tool-boundary context rendering", which card 12 removes.

### [NIT] The prepare_test.go header goes stale

**Location:** `_lym/plan/16-claude-trust-sentence.md`; `internal/shuttleengine/claudeengine/prepare_test.go:3`

**Issue:** The header names "the --append-system-prompt notice", which card 16 replaces with `--append-system-prompt-file`.

## Rejected

- guard-reach finding 1, the line cap refusing program notices other than LOOM's: the added decision "A post body is capped at the hook field budget" caps every post, and D1 singles out only LOOM's decision notice for special handling.
  The plan carries the decision record as written.
  The affected-set lens found the orch, batten and resume-line posters well under 8 KiB.
  Any remaining case is a question about the decision record, not a defect of the plan.
- way-forward NIT 5, the decision-notice refusal not naming the title: `decisionWayNoticeTooLong` follows D1's way forward verbatim ("shorten the decision or rationale and put the detail in a file").
  A title that alone pushes a notice past 8 KiB is not a realistic input.
- Card 13's subagent Stop exiting 1, against D3's "exits 0": D2 fixes a Stop's exit at 0 exactly when a block was written.
  Settings hooks fire `SubagentStop` and not `Stop` for a subagent (Constraints), so the card follows D2 for a guard case and states why.
- Card 4 keeping `maxBodyBytes` at 64 KiB: the overview's Shared Decisions shows that the line cap makes every post refuse a body past the budget, as the added decision asks.
  It also keeps the acceptance criterion that a reply to EXTERNAL over 8 KiB is accepted.
