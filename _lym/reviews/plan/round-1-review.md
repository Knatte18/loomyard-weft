---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/06-hook-wait-hands-mail.md (Tests); evidence internal/creelcli/cli_test.go:1011
    summary: Card 6 leaves the TestCreelCLI subtest for a superseded hook-wait asserting the newer waiter's stderr is BellLine, which card 6's own change falsifies, so its tier1 batch gate fails
    fix: In card 6's Tests list add a bullet that the tier1 TestCreelCLI subtest "a hook-wait superseded by a newer one of the strand exits 0 printing nothing" changes its newer-waiter assertion from creelengine.BellLine plus a newline to hookLine(1, alpha, "mail") + "\n" + hookLine(2, alpha, "more mail") plus a newline, with exit 2 unchanged.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: contracts/stencils/orch/orch-template-role.md:40, covered by no card
    summary: The orch stencil still tells an idle orch it is woken by the bell line and should end that turn before the mail shows, which D3 and D4 make false; no card lists it
    fix: In card 6 add `contracts/stencils/orch/orch-template-role.md` under **Edit:** (checked with validate-plan --overlay, no new finding) and add to its Intent that step 4's sentence becomes that an idle orch is woken with its mail under `[creel] Mail that arrived while you were idle:` and acts on it in that turn, and that only an orch no waiter reaches gets the fixed bell line, after which the mail arrives as that turn ends; the sentence before it adds the wake to where a mail line arrives.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/04-ring-claims-mail.md (last Intent paragraph) and _lym/plan/06-hook-wait-hands-mail.md (doc sentence)
    summary: D3's bounds besides the pause window, and the delivered-once constraint's two exceptions, are documented by no card, though the Scope puts the docs that state these bounds in scope
    fix: Extend card 4's internal/creelengine/doc.go Idle wake list with D3's other Bounds - the mail is journaled read before Claude Code shows it, so a session that dies or a hook ended at its timeout between the read and Claude Code reading the exit loses it as read, recorded by the acknowledged log line; a refused or lost confirmation shows the unjournaled mail twice, once in the wake and once at a later claim, and the bell may follow the woken turn; Claude Code escapes system-reminder tags and its marker patterns in the wake body - and have the claim paragraph's delivered-once sentence name the two exceptions of the Constraint (hook-read, and hook-wait once its acknowledgement is written, show mail whatever the acknowledgement's outcome; the ring's mail is journaled before it is shown).
  - id: F4
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-ring-claims-mail.md (Ring paragraph)
    summary: ringHook decides whether to read the pause from the waiter registered before its locked step, so a marked waiter that supersedes an unmarked one in between is rung with a claim no pause read covered
    fix: In card 4's Ring paragraph state that the locked step claims only when the pause was read for a marked waiter before it, and that a marked waiter found in the locked step without that read rings nothing and stays registered, so the next probe reads the pause.
  - id: F5
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/04-ring-claims-mail.md (Protocol and Ring paragraphs)
    summary: Card 4 adds the takes_mail mark and the ring's claim path without the Intent sentence on what the new edge lets through and what bounds it
    fix: Add to card 4's Intent a sentence stating that the mark lets a strand's own idle waiter take that strand's claimable mail, the set the claim verb would hand the same strand, journaled read at its acknowledgement and skipping only the Stop cap, which the ring starts over anyway; bounded by HookWait's refusal of every sender but a strand, the pause read, the hold, hookFieldCap and the 8 KiB budget.
  - id: F6
    severity: LOW
    class: scope
    origin: handler
    location: _lym/plan/03-ack-failure-releases.md (Intent, third paragraph)
    summary: Card 3 extends the release on a failed journal write to the read verb, which D3 does not license, since D3 names the ring or any other claim, and the delivered-once Constraint lists no read exception
    fix: Either confine card 3's release to claim and ring readings, leaving a read's reading as it is today, or ask the parent to record the read verb's change in the decision record before keeping the sentence about the read verb in card 3's Intent.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/09-claim-delivery-log.md (released bullet)
    summary: Reading.Release logs released unconditionally, so daemon.handOut's Release after a failed write of an empty claim logs a second line after the Debug nothing line, against D5's one line per attempt
    fix: In card 9's released bullet state that Reading.Release logs only when the reading held ids or echo items, and add a TestStore_ClaimLog row releasing an empty claim reading that asserts the Debug nothing line alone.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/07-ring-echo.md (row ThePausedOrHeldStrandsClaimStillHandsOutTheEcho)
    summary: The row wants the held claim to hand out the same echo after the paused claim, but card 7 drops a claim's echo at its Ack, and the row does not say whether the paused claim is acked or released
    fix: State in the row that the paused claim is released, so the echo stays owed and the held claim hands it out again; or split it into two rows, each starting from its own acked ring.
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/10-smoke-hook-delivery.md (subtest on a Stop claim of more than five lines)
    summary: The new tier3 subtest wants no second copy of any token anywhere in the pane, which the model's own reply or tool call quoting a token breaks, so the check fails on behaviour D1 does not govern
    fix: Narrow the assertion to the stop-hook summary - the first post's token appears once in its Stop hook feedback lines and no Stop says line or second Stop hook feedback line carries any token - instead of counting the token anywhere in the pane.
  - id: F10
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/plan/01-hook-output-seam.md (Tests paragraph)
    summary: Card 1 says TestHookContext pins seven HookOutput rows and then lists six
    fix: Replace "seven HookOutput rows" with "a HookOutput row for each of" before the list in card 1's Tests paragraph.
---

### [MEDIUM] Card 6 misses the superseded hook-wait subtest

**Location:** `_lym/plan/06-hook-wait-hands-mail.md`, Tests; evidence `internal/creelcli/cli_test.go:981-1014`.

**Issue:** The subtest `a hook-wait superseded by a newer one of the strand exits 0 printing nothing` asserts at line 1011 that the newer waiter wrote `creelengine.BellLine+"\n"`.
After card 6, that waiter writes the claimed lines of #1 `mail` and #2 `more mail`.
Card 6 changes only the `rings a strand as exit 2` subtest and adds one, so `TestCreelCLI` fails at card 6's tier1 gate.
Both the affected-set and repo-invariants lenses reported it.

### [MEDIUM] The orch stencil keeps the bell-first wake

**Location:** `contracts/stencils/orch/orch-template-role.md:39-40`.

**Issue:** Step 4 tells the orch that an idle orch is woken first by `[creel] Mail is waiting for you.` and should end that turn, after which the mail shows.
Under D3 and D4, the waiter wakes the idle orch with the mail itself, under `[creel] Mail that arrived while you were idle:`, and the model is to act on it in that turn.
An agent told to end the woken turn works against the Goal's "no empty turn".
The bell remains only for a strand no waiter reaches.
CLAUDE.md puts what agents must be told into the stencils lym ships, so this follows D3 and D4.
An overlay adding the stencil to card 6's targets reported no new finding.
Both the affected-set and repo-invariants lenses reported it.

### [MEDIUM] D3's bounds and the delivered-once exceptions reach no doc

**Location:** card 4's `internal/creelengine/doc.go` list; card 6's `internal/creelcli/doc.go` sentence.

**Issue:** Scope puts in "the package docs and code comments that state these renderings and bounds".
Card 4's doc list carries D3's pause-window Bound alone.
Card 7 carries D6's Bounds, card 2 D4's, and card 9 D5's.
No card documents three of D3's Bounds: the loss window between the journaled read and Claude Code reading the exit, the double showing after a refused or lost confirmation, and Claude Code's tag escaping in the wake body.
No card states the Constraint's two exceptions to delivered-once either.
Card 3's doc sentence covers only the released ids coming back.

### [LOW] Pause read can miss a waiter that supersedes in between

**Location:** `_lym/plan/04-ring-claims-mail.md`, Ring paragraph; evidence `internal/creelengine/verbs.go:395-455`.

**Issue:** The pause is read only when the registered waiter takes mail, decided before the locked step.
`HookWait` supersedes inside the store lock, so a marked waiter that replaces an unmarked one in that gap is claimed for with no pause read.
It takes mixed binaries, which only a deploy produces.

### [LOW] The ring path's Intent states no attack-surface bound

**Location:** `_lym/plan/04-ring-claims-mail.md`, Protocol and Ring paragraphs.

**Issue:** The rubric asks a card adding a flag or routing edge to say in its Intent what the edge lets through and what bounds it.
Card 4 lists the boundary rules the claim keeps.
It never says the mark lets a strand's waiter take and journal that strand's mail outside a hook claim, never says that this skips only the Stop cap, and never names HookWait's strand-only refusal as the bound on who sets the mark.
Card 7's echo states its own: it continues no turn and shares the budget.

### [LOW] Card 3 widens the release to the read verb

**Location:** `_lym/plan/03-ack-failure-releases.md`, Intent.

**Issue:** D3 releases unjournaled ids "on the ring or any other claim".
D5 uses "claim" for the claim verb and the ring.
Card 3 states that the change covers a `read`'s reading too.
`lym creel read` would then hand out once more mail its client already printed, a duplicate outside the Constraint's two exceptions.
Today that mail stays reserved until a restart.

### [LOW] Release logs a second line for an empty claim

**Location:** `_lym/plan/09-claim-delivery-log.md`, `released` bullet; evidence `internal/creelengine/server.go:455-458`.

**Issue:** `handOut` calls `Release` when its write fails, even for a reading with nothing in it.
`ClaimAtBoundary` has already logged that claim's Debug `nothing` line, so the attempt gets two lines.

### [LOW] Card 7's paused-or-held row leaves the paused claim's end open

**Location:** `_lym/plan/07-ring-echo.md`, row `ThePausedOrHeldStrandsClaimStillHandsOutTheEcho`.

**Issue:** A claim's Ack drops its echo, so the held half sees the same echo only if the paused claim was released.
The row does not say which, so an implementer who acks it writes a failing row.

### [LOW] The five-line smoke check counts tokens anywhere in the pane

**Location:** `_lym/plan/10-smoke-hook-delivery.md`, new subtest.

**Issue:** The model acts on the seventh post and may quote any token in its reply or tool call.
"No second copy of any token" across the pane then fails on model output, not on the operator copy D1 governs.
D1's acceptance is about the stop-hook summary row.

### [NIT] Card 1's TestHookContext row count

**Location:** `_lym/plan/01-hook-output-seam.md`, Tests.

**Issue:** The text says seven `HookOutput` rows and lists six: three per event.
The tally also pins a number the list already gives.

## Rejected

- Card 1 names `capabilities.go`'s file header in its Intent with no file target: card 1 already targets members of that file, and `redundant-file-target` refuses the file beside them, so the header edit rides with the member edits.
- `pattern/PATTERN-daemon-wakeups.md`'s reconnect sentence for hook-wait: it stays true until a ring reaches the handler, as the repo-invariants lens found, so no card must change it.
- `AMarkedRingStartsTheStopCountOver` as a duplicate of `ARingStartsTheStopCountOverWhereTheDaemonStartCapRefusedMail`: it is a row in the same test, and it pins the marked ring's reset, a separate branch in the new locked step.
- Card order across cards 3 to 8: each `plan:` handle a card edits or uses (`FailJournalWrites`, `claimLocked`, `awaitAck`, `ErrAckUnsent`, `EchoHeading`, `HookOutput`) is created by an earlier card.
