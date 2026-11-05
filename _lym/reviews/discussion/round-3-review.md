---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:48
    summary: "D2's bound that no pane process can repoint the channel or operator id is false, since `lym config slack` writes those machine-layer keys from the prime orch's pane and its guard is a mistaken-write guard only"
    fix: "In D2, keep the channel id and the operator's Slack user id in `slack.json` beside the token, written only by `lym slack setup` after its `conversations.info` and `users.info` checks, and leave only `remind_after_min` in the new `slack` config module; change the relay's per-poll read in D2 to read those two ids from `slack.json`, and the D3 `lym slack status` and D4 new-channel path wording to match. If the ids stay in config instead, replace the second D2 Bound line with the true one: the prime orch's pane, and any process that clears `LYM_STRAND_NAME`, can still repoint channel and operator through `lym config slack --layer machine`, a repoint reaches the live relay at its next poll with none of setup's checks, and name what bounds that (the relay posts only to a channel its bot is a member of, and only the configured user's text is forwarded). In both cases add to the D2 acceptance bullet that a `lym config slack` write of the channel or operator id from a pane does not repoint a live relay, or states the residual path."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:44
    summary: "D2, D3 and D6 pane-ancestry refusals do not say what the check does while a live strand's pane process is unknown, so the guard on login, setup, start, away, back and stop can fail open"
    fix: "In D6's pane-ancestry bullet, state that the check refuses, as creel's sender resolution does, when any live strand's pane process is unknown, because the caller could be that strand, and that the refusal names waiting until every live strand's pane process is known as the way forward beside the operator's terminal. Make D2's refusal bullet and D3's `lym slack start` bullet refer to that same check, and add to the D2 and D6 acceptance bullets that a verb run while a live strand's pane process is unknown refuses and writes or changes nothing. If fail-open is the intent instead, state in each of D2, D3 and D6 what the window lets through and what bounds it."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:143
    summary: "The hidden `lym slack relay` verb has no stated bound, so a non-orch strand running it by hand can hold the single-instance lock and block the real relay, and the relay's reaction to a refused first read is unstated"
    fix: "In D3, state that the relay ends, releasing its lock, when its first creel read is refused, after writing creel's refusal to its log in `.lym/slack/` where `lym slack away` names it, and that a later refused read is retried with the poll's backoff. State that the first read comes before any Slack poll, so `lym slack relay` run by hand from a non-orch strand's pane, which creel refuses as a program claim from a non-orch pane, polls Slack for nothing and leaves the lock free. In D9 say `relay` carries no operator audience beyond the starter's double fork. Add to the D3 acceptance bullet that a `lym slack relay` run from a non-orch strand's pane exits after the refused first read, forwards nothing, and a later start takes the lock."
---

### [BLOCKING] D2's no-pane-repoint bound is false for channel and operator id

**Location:** `_lym/discussion/decision-record.md:40` and `:48`.
**Issue:** D2 puts the channel id and the operator's user id in `slack` machine-layer config and bounds the risk by saying no process inside a strand's pane can repoint them, because `lym slack setup` refuses from a pane, an orch's included.
`lym slack setup` is not the only writer.
`internal/configcli/hubwide.go:46-51` and `internal/configcli/doc.go:25` show that a write to a hub-wide module's machine layer is refused only from a non-prime worktree and from a session whose `LYM_STRAND_NAME` carries a slug; the source comment says the guard "guards a mistaken write, not a determined agent" and reads the strand name the caller passes and nothing else.
The prime orch's name is slug-free, so `lym config slack --layer machine` from the prime orch's pane writes both keys, which is the case D2 sets out to refuse for `setup`.
The relay reads the two keys at each poll, so the write reaches a live relay at its next poll with none of `setup`'s `conversations.info` and `users.info` checks, and D4's new-channel path then relays whatever channel it names.
Pointing the channel at another channel the bot belongs to sends the orchs' mail there; pointing the operator id at another user hands that user the operator's authority (D7).
Both are what the bound claims cannot happen.
The fix either moves the two ids to the credential file, where only `setup` writes them, or states the true bound.

### [MEDIUM] The pane-ancestry check's unknown-pane case is unspecified

**Location:** `_lym/discussion/decision-record.md:44`, `:57` and `:109`.
**Issue:** D2, D3 and D6 refuse `login`, `setup`, `start`, `away`, `back` and `stop` from a live strand's pane, "read from the pane ancestry as D6's check does", which D6 ties to creel's sender resolution.
Creel's resolution (`internal/creelengine/identity.go`, `paneOfPeer`) has a third outcome beside in a pane and in no pane: when a live strand's pane process is unknown, a process in no known pane cannot be told from that strand, and creel refuses.
The record borrows the two clean outcomes and says nothing of the third.
D4 and D6 already name that state as a real one, since they rely on creel refusing the relay's read in it.
A check that treats an unreadable pane as no pane lets a strand whose pane process is unknown run `lym slack login` and write the token file, or `lym slack setup` and repoint the channel, which is the bypass D2's bounds say is closed.
The attack-surface question needs the Decision to say which way the window goes and what bounds it.

### [MEDIUM] The hidden `lym slack relay` verb is unbounded

**Location:** `_lym/discussion/decision-record.md:143`, with `:53` and `:62`.
**Issue:** D9 lists `relay` as hidden and internal and gives it no audience or refusal, while D2, D3 and D6 bound every operator verb.
Nothing stops a non-orch strand from running `lym slack relay` in its own pane.
It takes the single-instance lock before D3's first creel read, and creel refuses that read as a program claim from a non-orch pane (`identity.go`, the `isOrchName` branch).
The record does not say that the relay then exits.
If it stays, it holds the lock without polling, a start that finds the relay live does nothing (D3), the prime dobby's ticks do nothing, and the operator's Slack messages stop with no refusal anywhere; `lym slack away` times out, naming a relay log that shows only the refusal.
The record also leaves open what the relay does after a refused read in general, which D6's `away` refusal and D4's wording both lean on.
Stating that a refused first read ends the relay, and that the first read comes before any poll, bounds the verb with no new refusal.

## Rejected

- D3's double fork against `PATTERN-reap-children` (focus 1): not a defect. `## PATTERN changes` lists `PATTERN-reap-children` with the declared text word for word, bar a double fork's intermediate, which exits at once; D3 names the starter's wait on the intermediate and calls the release of the relay the one exception that change admits. `lym` sets no child subreaper (a search of the Go sources finds none), so D3's stated assumption holds for lym's own processes, and a violation fails safe, since creel refuses the read and D6's `away` names the refusal.
- D5's handling of creel's 30-second acknowledgement deadline (focus 2): not a defect. `ackReadTimeout` is 30 seconds in `internal/creelengine/server.go`, and the deadline starts when the messages are handed out. The client's `Call` acknowledges only when the handler returns nil and a handler error leaves the read released (`internal/creelengine/client.go`), which is the stop-and-release path D5 states. D5 clears the posted-id record only after the daemon's confirmation, keeps it on a refused or lost acknowledgement, and the next read skips the ids it holds, so a backlog that outlasts the window posts each message once; the acceptance bullet names that case. The stop margin is a Plan detail.
- D5's sender filter against `agentname.IsProgram` (focus 4): not a defect. `IsProgram` does report `<shortname>:EXTERNAL` as a program, and D5 excludes it by name, in the filter bullet, in the Bound line and in the acceptance bullet. Creel refuses a program claim named EXTERNAL and stamps EXTERNAL only in the two-segment form, so the scoped form needs no exclusion.
- D4's outcome for a forwarded message whose creel post fails (focus 5): not a defect. The cursor stays before the message, the failure is logged at Warn, the retry follows the poll's backoff, and after 5 minutes the relay answers in Slack and moves on, so the retry is bounded and nothing is skipped silently.
- Attack-surface bounds on `lym slack login`, `lym slack setup` and `lym slack start` (focus 3): the stated refusals and the token's own Bound lines are sufficient except for the two gaps raised as F1 and F2.
