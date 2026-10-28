---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/11-report-replies.md
    summary: ComputeReplies imports internal/creelengine and internal/agentname, which internal/reportengine's import allowlist lacks, so card 11 fails TestToldGeometryInvariant_AllowlistOnly on its own tier0 gate
    fix: >-
      In _lym/plan/11-report-replies.md add a `**Extends:**` group before `**Uses:**` holding
      `internal/reportengine#allowedImports`, and add one Intent sentence: `allowedImports` in
      internal/reportengine/seam_enforcement_test.go gains `github.com/Knatte18/loomyard/internal/creelengine`
      and `github.com/Knatte18/loomyard/internal/agentname`, the two packages `ComputeReplies` imports,
      so `TestToldGeometryInvariant_AllowlistOnly` passes. The combined overlay run admits this target.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/07-chase-clock.md
    summary: The chaser probes the holder through the delivery seam, which runs reed status and so tmux, at a fixed one-minute wait that never backs off, also while a dormant, paused or held holder stops the clock indefinitely, breaking the Backoff rule of PATTERN-daemon-wakeups
    fix: >-
      In _lym/plan/07-chase-clock.md add the Create bullet
      `plan:internal/creelengine#chaseProbeCeiling` -> `const chaseProbeCeiling = 8 * time.Minute`
      (the combined overlay run admits it) and rewrite the clock paragraphs: while the clock runs the
      chaser ticks every `agedPersistInterval` and charges as planned; a tick that reads the clock stopped
      doubles the next wait up to `chaseProbeCeiling`, and a tick that reads it running, or a wake of the
      holder, returns the wait to `agedPersistInterval`; time is charged only between two running ticks one
      `agedPersistInterval` apart, so a backed-off wait never charges stopped time. `isChaseWait` matches
      `agedPersistInterval <= d <= chaseProbeCeiling`, which stays clear of `isSweepWait`'s `8s < d <= 50s`.
      The chase-timer bullet card 7 adds to `pattern/PATTERN-daemon-wakeups.md` states the backoff.
      `TestScheduler_Chases` gains a row: a dormant holder's chaser waits double from one minute to the ceiling
      and drop back to one minute once the holder is live again, and no time is charged meanwhile.
      Reword the overview's Shared Decision `chase-clock-cadence` to match: one fixed wait while the clock runs,
      backing off to the ceiling while it is stopped.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/05-expect-reply-engine.md
    summary: Card 5 answers a strand or program reply wait only inside Reply, so a post --wait-reply that reconnects with wait-reply after its connection dropped never gets a reply that landed meanwhile, and since the message is then replied and never chased the wait hangs to its timeout or forever
    fix: >-
      In _lym/plan/05-expect-reply-engine.md add `internal/creelengine#Store.settledOutcomeLocked` to the
      `**Edit:**` list (the combined overlay run admits it) and add an Intent bullet: its reply-wait case
      also answers a wait on a message whose sender is not EXTERNAL with `OutcomeReply` carrying the oldest
      reply whose `ReplyTo` is the id, so a reconnecting `--wait-reply` gets a reply that landed while it was
      away; card 7's unanswered case comes after it. `TestStore_Mail` gains a row: a strand's `PostWait` is
      cancelled, the holder replies, and a `WaitReply` registered afterwards is answered `OutcomeReply` with
      that reply.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/07-chase-clock.md
    summary: Only a chase posted record skips the known-mailbox mark, so the first CREEL sender notice makes the CREEL program a receiver anyone can post to, where mail sits unread forever, and card 8's notice-reply rationale that CREEL holds no mailbox becomes false
    fix: >-
      In _lym/plan/07-chase-clock.md change the Journal-and-fold bullet so `applyPosted` leaves `programs`
      alone for every `posted` record whose sender's program name, read with `agentname.ParseProgram`, is
      `ChaserProgram`, a chase and a sender notice alike, so `CREEL` never becomes a receiver.
      `TestJournal_Fold` gains a row folding a sender notice and asserting `CREEL` is not a known mailbox.
  - id: F5
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/00-overview.md
    summary: Shared Decisions former-holder-reply-guard and chase-forward-way-forward drop D10's second way forward and the reaching test the PATTERN-refusal-way-forward Constraint asks for, with no Added after Discussion entry recording either drop
    fix: >-
      Ask the parent to record, with `lym loom decision add`, an Added after Discussion entry amending D10
      and its Constraint: the refusal of a forward of a chase names only `lym creel post --reply <chase id>`,
      and the refusal of a chase reply from a former holder is a fail-loud guard with no reaching test, both
      because `Store.Forward` refuses a read message and a chase goes only to the holder of a read message.
      Once the record carries the entry, cite it in the overview's Shared Decisions
      `former-holder-reply-guard` and `chase-forward-way-forward` and in card 8's Intent. Card 8's code
      design stays as written.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/06-expect-reply-cli.md
    summary: Card 6 introduces the --expect-reply flag with no statement of what it lets a sender set in motion or what bounds it, which the attack-surface rule asks of the card that introduces a flag
    fix: >-
      In _lym/plan/06-expect-reply-cli.md add an Intent paragraph: Bound: the flag adds no privilege, since
      it marks a post the sender could already make to a receiver the daemon's usual rules admit; what the
      mark sets in motion, from card 7 on, is at most two content-free `CREEL` chases to the holder and one
      notice to the sender per marked message, and only once the holder has read it.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/05-expect-reply-engine.md
    summary: Making LineFits measure the marked line also tightens internal/loomcli noticeFits, which judges unmarked lym loom decision add notices, and card 5 neither lists that caller nor says the tighter bound is intended
    fix: >-
      In _lym/plan/05-expect-reply-engine.md add a `**Uses:**` group holding `internal/loomcli#noticeFits`
      (the combined overlay run admits it), and state in the Intent and the ImpactSummary that `noticeFits`
      now measures a decision notice against the marked form, so a notice within the mark's length of the
      8 KiB limit takes the record-pointing form, a conservative bound no notice post can exceed.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/09-sweep-holder-gone.md
    summary: Card 9 widens the removed-holder sweep, but the sweep bullet of pattern/PATTERN-daemon-wakeups.md still says it runs while mail or injects exist, and neither card 9 nor card 7 rewrites it
    fix: >-
      In _lym/plan/09-sweep-holder-gone.md add `pattern/PATTERN-daemon-wakeups.md` to the `**Edit:**` list
      (the combined overlay run admits it) and add one Intent sentence: the creel removed-holder sweep bullet
      says the sweep runs while a strand holds unread mail, a queued inject or a read, marked, unanswered
      message.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/12-stencils-through-creel.md
    summary: After card 12 the seat smoke fixtures ask and answer through creel, but internal/seatengine/smoke_seats_test.go still says the chair asks over the session channel and that its SendMessage starts the advisor's turn
    fix: >-
      In _lym/plan/12-stencils-through-creel.md add an `**Edit:**` group holding
      `internal/seatengine_test#copyTestdataStencil` and `internal/seatengine_test#TestSmokeSeatsChairAsksAdvisor`
      (the file header at lines 3-4 sits outside any member, so `edit-test-file` asks for every member of the file),
      an Intent sentence that the file header and the comment at lines 125-126 name creel mail in place of the
      session channel and `SendMessage`, and the `**ImpactSummary:**` line an Edit group requires: stencil and
      fixture wording and the seat smoke test's comments, with no Go behavior change. The solo and combined
      overlay runs admit this shape.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/07-chase-clock.md
    summary: Card 7 carries a program in readers only when its read lies in a file past retention, but plans readers as a set with no day, so that condition cannot be decided
    fix: >-
      In _lym/plan/07-chase-clock.md plan `readers` as a map from each program to the journal day of the read
      record that put it there; `carryRecords` writes a carried read record for each reader whose day is before
      the cutoff, `applyCarried` folds that record back into `readers`, and `moveOrigins` moves the day as it
      does a message's origin. `TestJournal_Retention`'s new row also asserts that a program whose only read is
      past retention is still in `readers` after the carry.
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/05-expect-reply-engine.md
    summary: The TestStore_Mail rows asserting Replied cannot observe it, since that external test sees a read message through no exported surface, while the internal TestJournal_Fold row already covers Replied
    fix: >-
      In _lym/plan/05-expect-reply-engine.md drop the two `TestStore_Mail` rows that assert `Replied` (a reply
      by the holder, a reply by the receiver a marked message was forwarded to), keeping the `ReplyTo` log-entry
      assertion, and extend the `TestJournal_Fold` row to also fold a forwarded record and a reply by the new
      holder, asserting `Replied` there.
  - id: F12
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/02-creel-post-parent.md
    summary: Card 2 plans four TestCreelCLI resolution rows whose walks TestAsker from card 1 and orchcli TestParentOrchStrand already cover, where one row pins the --parent wiring
    fix: >-
      In _lym/plan/02-creel-post-parent.md keep one resolution row in `TestCreelCLI`, from the beam `b` with its
      orch live, which runs `orchroute.Asker`'s beam branch through the CLI and asserts the envelope's `receiver`
      and the holder, plus the three refusal rows; drop the rows from `p`, from `m` with `b`'s orch live and from
      `m` with it not live, whose walks `TestAsker` (card 1) and `internal/orchcli#TestParentOrchStrand` cover.
---

### [BLOCKING] reportengine's import allowlist misses card 11's imports

**Location:** `_lym/plan/11-report-replies.md`, `plan:internal/reportengine#ComputeReplies`

**Issue:** `ComputeReplies(entries []creelengine.LogEntry)` imports `internal/creelengine`, and its grouping calls `agentname.Parse` and `agentname.ParseProgram`.
`internal/reportengine/seam_enforcement_test.go` pins production imports to `allowedImports` through `scankit.AssertImportAllowlistNoStale`, and neither package is on the list.
The list's own comment says each later engine card extends it.
Card 11 names neither the allowlist nor the test, so the card fails its own tier0 gate.
The fork's class was `consistency`; it is kept as `scope`, a missed site.

### [BLOCKING] The chase timer never backs off

**Location:** `_lym/plan/07-chase-clock.md`, the clock paragraphs; overview Shared Decision `chase-clock-cadence`

**Issue:** Every chaser tick probes the holder "through the delivery seam".
In production that is `hubDelivery.Strands`, which runs `reedengine.Engine.Status` per worktree (`internal/creelcli/delivery.go:154-300`, `internal/reedengine/lifecycle.go:1327-1342`) and so spawns tmux, and then `Paused`.
Card 7 fixes the wait at exactly `agedPersistInterval` and ends the chaser only once the holder holds nothing chaseable.
A dormant or paused holder, or one under another sender's hold, stops the clock with no end in sight.
The chaser then keeps a once-a-minute process-spawning probe for as long as that lasts, while nothing changes.
`PATTERN-daemon-wakeups`' Backoff rule says such a wake doubles its wait from a base to a named ceiling while nothing changes.
D10's bound ("wakes at its next due step") also gives no licence for a fixed probe on a stopped clock, because a stopped clock has no next due step.
Card 7 also lists the loop in `pattern/PATTERN-daemon-wakeups.md`, which would put a loop that breaks the rule onto the rule's own list.
The fix keeps the minute grid while the clock runs, which D10's `chase-aged` persistence needs, and backs off only while the clock is stopped.

### [MEDIUM] A reconnecting --wait-reply misses a reply that landed meanwhile

**Location:** `_lym/plan/05-expect-reply-engine.md`, the `Reply` bullet

**Issue:** `Client.resume` turns a dropped `post --wait-reply` into `VerbWaitReply` (`internal/creelengine/client.go:180-181`).
`settledOutcomeLocked` hands over a standing reply only when it is undelivered and held by EXTERNAL (`internal/creelengine/pool.go:377-388`).
Card 5 makes `Reply` answer a strand's or program's wait only at that moment.
A reply that lands while the waiting client is reconnecting is therefore never handed over.
The message is then `Replied`, so card 7 never settles it `unanswered` either, and the wait hangs.

### [MEDIUM] A CREEL sender notice makes CREEL a receiver

**Location:** `_lym/plan/07-chase-clock.md`, the Journal-and-fold bullet; card 8's notice-reply paragraph

**Issue:** `applyPosted` marks every program sender's mailbox known (`internal/creelengine/fold.go:303-305`), and `checkReceiverLocked` admits any known program (`internal/creelengine/pool.go:203`).
Card 7 exempts only a chase's `posted` record.
The sender notice of an `unanswered` outcome is a `CREEL` post without `Chased`, so it makes `<shortname>:CREEL` a receiver.
From then on posts to it are accepted and are never read.
Card 8 justifies its notice-reply refusal by saying `CREEL` holds no mailbox, which this path makes false.

### [MEDIUM] Two Shared Decisions drop parts of D10 with no recorded decision

**Location:** `_lym/plan/00-overview.md`, Shared Decisions `former-holder-reply-guard` and `chase-forward-way-forward`; `_lym/plan/08-chase-reply-forward.md`

**Issue:** D10 names two ways forward for a refused chase forward, and its acceptance criterion and the `PATTERN-refusal-way-forward` Constraint ask that a test reach the former-holder reply refusal.
The plan names one way forward and writes the former-holder branch as a fail-loud guard with no test.
The substance holds.
The guard-reach lens confirmed that no path moves a read message: `Forward` refuses one (`internal/creelengine/verbs.go:551`), and the sweep and `Withdraw` move only unread mail.
The second way forward could never succeed.
Still, the rubric accepts an uncarried Decision step only through an `Added after Discussion` entry, the decision record has none for these, and the plan-writer stencil admits no other route for a drop.
So the departure needs the parent's record, not the plan's own say-so.
Shared Decision `prime-hosted-asker-bound`, the focus's other departure, drops nothing.
D1 and D3 are followed as written.
The prime-hosted strands it names, the tackler and the arbiter, render the no-parent directive (`internal/sheddriver/tacklerprompt.go:55`, `internal/shedarbiter/prompt.go:56`), so they never post `--parent`.

### [MEDIUM] --expect-reply states no bound

**Location:** `_lym/plan/06-expect-reply-cli.md`, Intent

**Issue:** Card 6 adds the operator-facing `--expect-reply` flag.
The flag is what makes creel post chases to the receiver and a notice to the sender.
Card 2's `--parent` and card 15's removal each state what they let through and what bounds it, but card 6 states neither.
Card 7's bound sentence sits on the engine card, not on the card that introduces the flag.

### [LOW] LineFits' new measure reaches decision notices unlisted

**Location:** `_lym/plan/05-expect-reply-engine.md`, the `MessageLine` and `LineFits` bullet

**Issue:** `internal/loomcli/decision.go:302-312` (`noticeFits`) judges `lym loom decision add` notices with `creelengine.LineFits`.
Those notices are never marked, so card 5's change tightens their fit by the mark's length.
The card does not list the caller or say that this is intended.
The fork rated it MEDIUM.
It is kept at LOW: the tightening is a few dozen bytes, it can only make a notice choose the shorter form earlier, and no assertion the lens found breaks.

### [LOW] The sweep bullet in PATTERN-daemon-wakeups goes stale

**Location:** `_lym/plan/09-sweep-holder-gone.md`; `pattern/PATTERN-daemon-wakeups.md`, the creel removed-holder sweep bullet

**Issue:** That bullet says the sweep "runs while mail or injects exist".
Card 9 widens what the sweep runs on, and card 7 only extends the file with the chase timer.
The affected-set and repo-invariants lenses reported this defect independently, and it is merged here.

### [LOW] The seat smoke test's comments still name SendMessage

**Location:** `_lym/plan/12-stencils-through-creel.md`; `internal/seatengine/smoke_seats_test.go:3-4,125-126`

**Issue:** Card 12 moves `smoke-chair.md` and `smoke-advisor.md` to creel mail.
The tier3 test that drives them still describes the chair asking "over the session channel" and the chair's `SendMessage` starting the advisor's turn.
The D13 scan skips `_test.go` files, so nothing would catch the stale text.

### [LOW] readers carries no day for retention

**Location:** `_lym/plan/07-chase-clock.md`, the `applyRead` and retention bullets

**Issue:** `carryRecords` is to carry each program in `readers` "whose read lies in a file past retention".
`readers` is planned as a plain set, while every other carried entry keeps an origin day that `moveOrigins` moves (`internal/creelengine/fold.go:451-468`).
As written, the condition cannot be evaluated.

### [LOW] TestStore_Mail cannot observe Replied

**Location:** `_lym/plan/05-expect-reply-engine.md`, Tests

**Issue:** `TestStore_Mail` lives in `creelengine_test` and sees messages only through `Read`, which hands out unread mail, and `Log`, whose entries carry no `Replied`.
Its two `Replied` rows have nothing at the public surface to assert, which leaves them redundant with `TestJournal_Fold`'s fold row.
`TestJournal_Fold` is internal, already covers a reply setting `Replied`, and can take the forwarded case too.

### [LOW] Card 2's resolution rows repeat the walk tests

**Location:** `_lym/plan/02-creel-post-parent.md`, `TestCreelCLI` rows

**Issue:** Card 2's four rows cover the pair from the prime, the member pair with its beam live and not live, and the beam itself.
Each resolves through the single `orchroute.Asker` call.
`TestAsker` (card 1) covers the beam, prime and member-pair-to-live-beam answers.
`internal/orchcli#TestParentOrchStrand` covers the pair walks through the delegation.
One CLI row pins the wiring, and the refusal rows pin the rest of the card's own behaviour.

## Focus verdicts

- Card order, cards 5 to 11: no card uses a symbol a later card introduces.
  Card 2's `--expect-reply` combination is stated as from card 6 on, card 9 uses only card 7's symbols, card 10 uses card 7's `OutcomeUnanswered` and card 11 uses card 5's and card 7's log fields.
- Reachability, cards 2, 8 and 10: every refusal of card 2 is reached by a named input and a `TestCreelCLI` row.
  Card 8's chase-forward refusal is reached by a holder forwarding the unread chase.
  Card 8's notice-reply refusal is reachable, though its premise needs F4.
  The former-holder branch is a correct fail-loud guard, with its recording gap in F5.
  Card 10 adds an exit, not a refusal, and its row reaches it.
- Attack surface: card 2 and card 15 state their bounds, and card 6 does not (F6).
- Fidelity: every Decision D1 to D13 and every Constraint is carried by a card, apart from the gaps F5 names.
  `prime-hosted-asker-bound` drops no Decision.
  Card 5's change that ends a strand's `--wait-reply` with its reply is licensed: D10's waiting-sender outcome presumes it, and the creel doc's Mail section already states it, while `Store.Reply` today answers only an EXTERNAL sender's wait.
- Test economy and granularity: F11 and F12.
  The rows of cards 7, 9 and 10 each pin a behaviour no other row covers, and the card split is one reviewable unit per card.
- PATTERN fit: card 16's `PATTERN-agent-messaging` line matches the declared text and its place.
  The cards touching `internal/creelengine` break `PATTERN-daemon-wakeups` (F2), and those touching `internal/orchroute` and `internal/creelcli` break no entry.
  No card touches `internal/shedadapters`.

## Rejected

- Repo-invariants noted that card 10's claim "no existing row posts a marked message" is false once `--wait-reply` implies the mark. The lens itself found it harmless, since those rows remove the receiver before it reads, so no chaser starts. It is no finding.
- Card 8's refusal of a reply to a `CREEL` notice is not unlicensed scope. It follows from D10's sender notice, which would otherwise let a reply sit undeliverable. Its premise is handled under F4.
- Card 4's `askerOrchStrand` row is not redundant with `TestAsker`. It pins that the hold-notice route resolves through the asker rule and not `parentOrchStrand`, which no other row checks.
