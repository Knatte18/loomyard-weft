---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:98-107
    summary: "D5 Fresh launch failure-path bound names only injects and TTL, but creel's TTL runs only while the receiver is live and unpaused, and unread mail from the old cycle has no bound at all"
    fix: "In D5's Fresh launch bullet, replace the sentence that begins 'Bound: until a retry succeeds, an old inject can still reach the new session' with the bound that holds: while the withdraw has failed and no retry has succeeded, an old ORCHWATCH inject, a /clear or /compact included, is typed on the relaunched session's first idle, unpaused probe, since the TTL counts only live unpaused time and does not delay that probe, and the old cycle's unread mail (the note request, a pause-resume line) is reminded to the relaunched orch until a retry succeeds, since mail has no TTL. Then either accept that exposure in one stated sentence, or add a Decision that lyx orch start does not launch or bind a session while the first withdraw is refused and reports the refusal with the way forward. Align the acceptance criterion 'A relaunch whose withdraw is refused gets the withdraw retried by the watcher' with the chosen bound, and say in the same bullet that the 'at most one line of the old cycle' bound holds for a successful withdraw only."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:114-117
    summary: "The turn-end gate reads each Stop's time from a stamp line, but the incremental event read consumes a stamp line before its payload exists, so that Stop reads with no stamp, never counts, and its step times out"
    fix: "In D5's 'Steps that wait on the session' bullet, add the rule for a stamp line separated from its Stop payload by a read boundary. The recording hook writes the stamp with one printf and the payload with a second command, and Runner.ReadEvents (internal/shuttleengine/session.go) consumes through the last complete line, so a read between the two advances the cursor past the stamp and the next read parses the Stop with no stamp. State that the claude engine's reading leaves a trailing run of stamp lines unconsumed until their payload line arrives, the way ParseSessionSignals in internal/shuttleengine/claudeengine/signals.go holds its consumed offset back, or that the Stop times come from that signal reading. Add an acceptance criterion with a stamp read in one tick and its payload in the next, which still counts."
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/discussion/decision-record.md:18
    summary: "Scope lists only that the claude event parser returns each Stop's time, but the mechanism also needs a time field on the neutral shuttleengine.Event, stamp pairing in a parser that today skips stamp lines, and the other engines' parsers"
    fix: "Rewrite the Scope bullet on the shuttle provider-seam change to list the changes: a time field on shuttleengine.Event (internal/shuttleengine/engine.go), the claude ParseEvents (internal/shuttleengine/claudeengine/events.go) pairing each Stop payload with its preceding stamp line and reading a missing or malformed stamp as the zero time, and the fake engine in internal/testkit/shuttlefake/shuttlefake.go setting the time so a cycle test against it can confirm a step. Say that the orchengine Session seam passes the field through unchanged. Name the stamp-pairing rule in D5 as the single place that defines 'a Stop with no readable stamp never counts'."
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/discussion/decision-record.md:138-143
    summary: "D6 retries as a program only on creel's refusal for running in no strand pane, but creel returns every refusal as a plain error string, and neither Scope nor any Decision adds a way for the caller to tell that refusal apart"
    fix: "Add to D6 and to Scope how NotifyPrime recognises the one refusal that triggers the program retry. The client returns errors.New(resp.Error) for every daemon refusal (internal/creelengine/client.go), so the caller can only match message text today. Pick one and list it in Scope: a typed refusal code on the creel response that the Go client surfaces as an exported error value, used by NotifyPrime with errors.Is, in internal/creelengine/protocol.go and client.go. Add the typed code to the creelengine and creelcli package docs named in Scope, and make the acceptance criterion on the program-name retry cover a detached caller whose spawner has exited."
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:143
    summary: "A notice refused by creel is warned and dropped and D8 removes the notice log, so a dropped notice has no durable copy; the drop list also omits a non-orch pane caller with no strand claim, which creel refuses for claiming a program"
    fix: "In D6, state in one sentence that a dropped notice survives only in the caller's warning, and require that warning to carry the notice's full line and the refusal, so the operator or the orch role can recover it from the logger. Add the case of a caller in a non-orch pane whose environment carries no strand claim (posted as its program, refused because only an orch pane may claim one) to the drop cases beside the cross-strand claim and the unresolvable-senders refusal, or state that the module constants' callers always run with a strand claim. Add a line to D8's fallback saying a notice creel refused is not in lyx creel log."
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:23-27
    summary: "Turn 4 and ledger Q7 describe the notice sender rule as strand when the caller runs in that strand's pane, but D6 now has the caller walk no ancestry and retry as a program on creel's refusal"
    fix: "In support-log.md Turn 4, replace the sentence 'Round 1 narrowed it to a caller running in that strand's pane, since a detached caller inherits the name but runs in no pane.' with the current rule: the caller claims the strand named by the environment, and creel's refusal for running in no strand pane triggers one retry as the program (D6). In the Q7 ledger line, replace 'strand when a non-orch strand and the caller runs in its pane, else program (D6)' with 'strand claim when the environment names a non-orch strand, one program retry on creel's no-strand-pane refusal, else program (D6)'."
---

### [MEDIUM] D5 Fresh launch failure-path bound covers neither the TTL's real meaning nor the old cycle's mail

**Location:** `_lyx/discussion/decision-record.md:98-107`

**Issue:** The first focus question was whether any message or inject the old cycle left queued or unread for the fixed orch name can still reach the relaunched session, and whether the stated bound covers both the withdraw's reach and its failure path.
The reach on the success path is bounded as stated: the withdraw removes only ORCHWATCH's queued injects and unread mail to one receiver, and a `sending` inject is typed to the end.
The failure path is not.
The decision bounds it with "for at most that TTL", but creel's doc (`internal/creelengine/doc.go:97`, `:122`, `:158`) says an inject's TTL "runs only while the receiver is live and no pause covers it".
A relaunch removes the strand first, so the old inject's clock is frozen until the new session is live.
When the new session comes up idle, creel's first probe types the old `/clear` or `/compact`; the TTL does not delay it.
So if the withdraw keeps being refused (the sender-resolution refusal of D1 is the stated cause), the old inject lands on the relaunched session within a probe interval, not within ten minutes.
The sentence is a ceiling on the exposure, not a bound on it.
The bullet also names only injects.
The withdraw also removes unread mail, and mail has no TTL.
When the withdraw fails, the old note request stays unread, the relaunched orch is reminded of it and writes a note nothing waits for.
The acceptance criterion "is reminded of no note request from the old cycle" cannot hold on that path, and the failure-path criterion speaks only of `/clear`.
This is the attack-surface question: a retry path that lets an old `/clear` through needs a stated bound, and the stated one does not hold.

### [MEDIUM] A Stop's stamp can be lost between reads, so the step's own turn end never counts

**Location:** `_lyx/discussion/decision-record.md:114-117`

**Issue:** The third focus question was whether the event data the turn-end gate relies on exists and at what precision.
The stamp exists: `recordingCommand` in `internal/shuttleengine/claudeengine/settings.go` writes `{"lyx_stamp":"Stop","lyx_at":"<RFC 3339 UTC>"}` with `date -u +%Y-%m-%dT%H:%M:%SZ` before each payload, so the precision is whole seconds, as the decision says.
The mechanism that reads it is not sound.
The stamp is one `printf` and the payload is a second command (`; cat >> …`), so two lines land a moment apart.
`Runner.ReadEvents` consumes "up to the last complete line" (`readEventsFrom` in `internal/shuttleengine/wait.go`), so a poll that falls between the two lines moves the cursor past the stamp and then reads the Stop alone.
A Stop with no readable stamp "never counts", so the step's genuine turn end is lost and the step times out.
`ParseSessionSignals` handles exactly this by holding its consumed offset before a trailing run of stamp lines (`signals.go`); the decision does not use it or say what replaces it.
The window is short, but it fails silently at the one signal the skills, retry and pointer steps advance on.

### [MEDIUM] Scope omits the shuttle changes the Stop time needs

**Location:** `_lyx/discussion/decision-record.md:18`

**Issue:** The Scope bullet says the claude engine's event parser returns each Stop's time.
`ParseEvents` returns `shuttleengine.Event`, which has no time field (`internal/shuttleengine/engine.go:84`), and the parser skips every stamp line and works one line at a time, so it has no stamp to pair.
The change needs a neutral field on `Event`, a pairing step in the claude parser, and the fake engine (`internal/testkit/shuttlefake/shuttlefake.go`) setting the time: with the zero time every Stop from the fake never counts, so the clear-cycle acceptance criterion cannot pass against it.
The plan writer reads Scope as the list of code changes, so the missing items would be found only at compile time or in a failing cycle test.
Call-site enumeration is not the finding here; the missing neutral type change and the fake are.

### [MEDIUM] D6's program retry depends on telling one creel refusal from the others, and nothing provides it

**Location:** `_lyx/discussion/decision-record.md:138-143`

**Issue:** The second focus question was whether the sender rule depends only on facts the caller can know.
The rule's inputs are sound: the environment's strand name and the refusal.
Against `resolveSender` (`internal/creelengine/identity.go`), a detached caller whose spawner has exited reaches no pane, and creel lists the strand it names, so creel refuses with "runs in no strand pane" and the program retry then resolves to a program sender.
A detached caller whose spawner lives still reaches the pane and signs as the strand.
A claim for an unlisted strand is signed EXTERNAL.
Those match D6.
The gap is the retry's trigger: the client returns every daemon refusal as `errors.New(resp.Error)` (`internal/creelengine/client.go`), and `resolveSender` returns plain `fmt.Errorf` values.
The caller can therefore only match message text, and the retry must fire on this refusal and not on the two refusals D6 says get none (a claim from another strand's pane, and the unresolvable-senders refusal).
No Scope item or Decision adds a typed code, so the one rule that separates "retry" from "drop" has no mechanism.

### [LOW] A refused notice is dropped with no durable copy, and one drop case is not listed

**Location:** `_lyx/discussion/decision-record.md:143`

**Issue:** The second focus question also asked whether any caller's notice can be refused and dropped.
It can, in the two cases D6 states: a claim naming one strand from another strand's pane, and every sender outside a pane while creel cannot resolve senders.
A third follows from `resolveSender`: a caller in a non-orch pane whose environment carries no strand claim is posted as its program and refused ("only an orch strand's pane may claim a program name"), and the retry fires only on the no-pane refusal.
D6 accepts the drop as "informational", but D8 removes the notice log in the same change, so a refused post is in neither the mailbox nor `lyx creel log`.
The orch role's fallback then cannot find it.
The warn-and-drop stays, but the decision should say where the line survives.

### [LOW] Support log still describes the earlier sender rule

**Location:** `_lyx/discussion/support-log.md:23-27`

**Issue:** Turn 4 ends with "Round 1 narrowed it to a caller running in that strand's pane", and the Q7 ledger line says "strand when a non-orch strand and the caller runs in its pane, else program (D6)".
D6 now has the caller walk no ancestry, claim the strand by environment, and retry as a program on creel's refusal.
The two notes record a rule the decision record no longer states.
