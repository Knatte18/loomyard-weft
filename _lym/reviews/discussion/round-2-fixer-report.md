# Round 2 fixer-report — discussion decision record

## Changes

All edits are in `_lym/discussion/decision-record.md`.

- **F1** (fixed as proposed, option a): added a `PATTERN-reap-children` "changes" bullet to `## PATTERN changes`.
  It keeps the rule and admits one exception, a double fork's intermediate that exits at once.
  D3's double-fork bullet says the starter waits on the intermediate and names this exception.
  Scope's In list names the exception too, which came from the self-check.
- **F2** (fixed as proposed):
  - D5 stops posting before creel's `ackReadTimeout` and releases the read with the rest unposted.
  - It waits out the backoff or a `429`'s `Retry-After` outside the handler.
  - It clears the posted-id record only after a confirmed acknowledgement.
  - D5's Bound says a refused or lost acknowledgement reposts nothing the record holds.
  - A new D5 criterion: a backlog that outlasts the acknowledgement window posts each message exactly once.
- **F3** (fixed as proposed, then bounded in the self-check):
  - D4: a failed or refused creel post leaves the cursor before that message, logs at Warn and retries with the poll's backoff.
  - After 5 minutes of failures the relay answers in Slack that the message was not delivered, quoting creel's refusal, and moves the cursor.
  - D4's Bound states the hold of at most 5 minutes.
  - D6's `away` refusal names creel's refusal of the relay's first read.
  - A new D4 criterion covers the refused post.
- **F4** (fixed as proposed):
  - D2: `login` and `setup` refuse from any live strand's pane, an orch's included, and name the operator's terminal.
  - A new D2 Bound says what remains open.
  - D3: `lym slack start` refuses from a non-orch strand's pane, by D6's check.
  - The D2 criterion now covers the login and setup refusals.
  - The D3 criterion now says start refuses from a non-orch pane. The identity check now applies to a relay start called from inside a non-orch pane, since the CLI verb itself now refuses there.
- **F5** (fixed as proposed): D5's sender filter and its criterion drop and do not post `<shortname>:EXTERNAL` mail.
- **F6** (fixed as proposed, first option): a prime anchor change starts the relay once it ends, whether it lands or fails and whatever dobbies it restarts. The D3 criterion already matches this wording.
- **F7** (fixed as proposed, first option): the D8 wait start is the later of dobby's first read and the `resumed_at` of the latest resumed pause-record entry covering the child. The D8 criterion now says "whether or not dobby read the pause".
- **F8** (fixed differently):
  - Proposed: a cursor keyed by channel id.
  - Made instead: the cursor carries the channel id, and a poll that finds a cursor for another channel takes the no-cursor path before reading.
  - Why: the self-check showed that a per-channel cursor replays history when setup moves back to a channel relayed before.
  - D2: the relay reads the channel and operator id from config at each poll.
  - The D4 criterion now covers a changed channel, a channel relayed before included.
- **F9** (fixed as proposed, Bound option): the re-exec bullet takes the lock before the next poll. A Bound says the loser of a race exits before its first poll, so only one relay polls.
- **Self-check consistency fix:** D5's cap bullet now waits out a `429`'s `Retry-After`, matching D4 and the new D5 line.

## Self-check

- fix-consistency, LOW, Scope omits the PATTERN-reap-children change: fixed as proposed.
- fix-consistency, LOW, D2 relies on a first-start path a live relay never takes: fixed with own change, a new-channel path at any poll (merged with the channel-return findings below).
- fix-consistency, LOW, D5's 429 wait disagrees with itself: fixed as proposed.
- fix-consistency, LOW, D4's Bound omits the hold behind a refused post: fixed as proposed, with the hold bounded to 5 minutes.
- fix-claims, LOW, a per-channel cursor replays history on a return to an earlier channel: fixed with own change, a single cursor carrying its channel (merged).
- fix-proposal, MEDIUM, a per-channel cursor replays history on a return to an earlier channel: fixed as proposed, keeping only the configured channel's cursor (merged).
- fix-proposal, MEDIUM, an unbounded creel refusal blocks presence commands: fixed as proposed, the head-of-line wait bounded at 5 minutes and then answered in Slack as not delivered (merged).
- fix-way-forward, MEDIUM, an unbounded creel refusal leaves the operator stuck on the phone: fixed by the same 5-minute bound and the not-delivered answer, which asks the operator to send the message again (merged).
- fix-way-forward, LOW, `away`'s refusal quotes a creel refusal that names no remedy: deferred, see below.

## Disputed

None.

## Deferred

- fix-way-forward LOW (`away` refusal offers no remedy): creel's refusal names the strand whose pane process is unknown, but no lym verb found in this round clears that state.
  The remedy belongs to reed or creel, outside this record's scope and its write surface.
  The refusal at least tells the operator why `away` failed and which strand is at fault.
