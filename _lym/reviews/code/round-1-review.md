---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/creelengine/verbs.go#Reply
    summary: A post --reply to mail from the hub's SLACK program skips the relay check, so a reply the relay drops or holds still answers plain ok
    fix: In Store.Reply, before journaling a local reply whose original.From is this hub's relay program (agentname.FormatProgram(s.geom.Shortname, BridgeProgram)) and whose sender is no program, call the same slackRelayCheck the post path uses. Return its refusal before anything is appended. Carry the live answer and relayDownWarning back to the caller by extending Reply's result and the VerbReply response in server.go and cli.go's --reply branch, so the CLI prints receiver_live and the creel warning line. Add rows for a refused reply (nothing journaled) and a down relay (warning) to TestStore_PostToTheSlackRelay.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/creelengine/verbs.go:227
    summary: No test pins that a post to any receiver other than the hub's relay program skips the bridge, so dropping the receiver guard would pass every test
    fix: Add rows to TestStore_PostToTheSlackRelay in internal/creelengine/machinemail_test.go, with the bridge refusing the sender and reporting the relay down. One row posts to a local strand and one to a beam's program such as ly:zz:SLACK. Each expects no error, the post journaled, and the receiver's own liveness (for the strand) or nil with no warning (for the program). Generalize the journal count's To filter to the row's receiver.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: scope
    location: internal/creelengine/verbs.go:212
    summary: A post stored while the relay is down answers receiver_live false and a warning instead of the delivered false field and reason the board entry names
    fix: Add a Delivered *bool field to PostResult and Response (json delivered, omitempty), set it in post beside ReceiverLive when slackRelayCheck answered, false with the reason when the relay is down, and print it in cli.go's post fields as delivered. Keep receiver_live and the warning line. Pin the field in the live and down rows of TestStore_PostToTheSlackRelay.
    origin: lens:code-scope
  - id: F4
    severity: LOW
    class: design
    location: internal/creelcli/slackbridge.go#RelayPostReady
    summary: A failed read of the geometry, beam listing, forwarding file or relay lock refuses the post with a raw error and no way forward, where it used to store mail the relay may still post
    fix: In RelayPostReady, wrap each read error from b.geometry, b.hasButton, slackengine.ForwardingOn and slackengine.RelayLive as a refusal that names what could not be read and ends in a way forward clause, such as run lym slack status, or report the text in your pane. Add a table row to TestSlackBridge_RelayPostReady with a failing geometry callback that asserts the way forward.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: design
    location: internal/creelengine/verbs.go#relayDownWarning
    summary: The relay-down warning promises the message waits until the relay starts, but the relay never posts mail older than a day
    fix: Reword relayDownWarning to say the message waits in its mail and the relay posts it once started within a day, and that older mail is dropped; keep the way forward clause naming lym slack start. Update the relayDown constant in TestStore_PostToTheSlackRelay to the new text.
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: design
    location: internal/creelengine/machinemail_test.go#TestStore_PostToTheSlackRelay
    summary: No row covers an EXTERNAL sender, which the docs and commit name as asked about, so a sender-kind filter that skipped EXTERNAL would pass
    fix: Add a row to TestStore_PostToTheSlackRelay with external() as sender and a bridge refusing ly:EXTERNAL, expecting the refusal and nothing journaled. Add a row to TestSlackBridge_RelayPostReady with sender ly:EXTERNAL expecting the error to contain ask your orch to post it.
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: consistency
    location: contracts/orchrole/role-slack.md:7
    summary: The orch's Slack role file still says a post to SLACK while the button is off is dropped, though such a post is now refused and a down relay now warns
    fix: In contracts/orchrole/role-slack.md, replace the button-off sentence with one saying a post to <shortname>:SLACK while the button is off is refused with nothing stored, and add one saying a post while the relay is down answers receiver_live false with a creel warning line naming lym slack start; on either answer the orch reports the text in its pane.
    origin: lens:code-affected-set
  - id: F8
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md
    summary: The change description lists the slack turn hook among program senders that keep their behaviour, but the hook posts as the orch strand and now passes through the bridge check
    fix: In _lym/landing/summary.md, drop the slack turn hook from the program-sender sentence and add that the hook posts as the orch strand after checking its own button, so it passes the new check and a down relay now gives it a warning instead of a plain ok.
    origin: handler
---

### [MEDIUM] A reply to Slack mail keeps the silent success

**Location:** internal/creelengine/verbs.go#Reply

**Issue:** The new check sits in `Store.post` alone.
`lym creel post --reply <id>` sends `VerbReply` (`internal/creelcli/cli.go`, the `replying` branch), and `Store.Reply` appends a local reply to `original.From` with no Slack check (`verbs.go`, `appendPostedLocked` for `To: original.From`).
Operator Slack text reaches the orch as mail from `<shortname>:SLACK` (`slackengine/inbound.go`, `in.mailbox.Post(orch, body)`), and the mail header offers the reply command.
An orch that answers that mail with `post --reply` while its button is off, or while the relay is down, gets a plain ok, which is the failure the board entry describes.

### [MEDIUM] The relay-receiver guard is unpinned

**Location:** internal/creelengine/verbs.go:227

**Issue:** Every row of `TestStore_PostToTheSlackRelay` posts to `ly:SLACK`.
The liveness test in `verbs_test.go` runs with no bridge, and the other `withBridge` tests assert no liveness on a local post.
If `receiver != relay` were dropped, every local post in a daemon with the bridge would be refused or have its liveness overwritten, and no test would fail.

### [LOW] `receiver_live` stands in for the named `delivered: false`

**Location:** internal/creelengine/verbs.go:212

**Issue:** The board entry allows a refusal, or an ok answer "with a `delivered: false` field and the reason".
For a down relay the diff answers ok with `receiver_live: false` and a stderr warning, and gives `receiver_live` a second meaning on this one receiver.
The sender is told, so the entry's purpose holds.
The field the entry names is absent, and the change description records the substitution as the writer's own choice, not as a parent decision.

### [LOW] A bridge read failure refuses with no way forward

**Location:** internal/creelcli/slackbridge.go#RelayPostReady

**Issue:** Each read error in `RelayPostReady` is returned raw, so `post` refuses the mail and stores nothing (`verbs.go`, `slackRelayCheck` returning the error).
Before the diff such a post was stored and the relay could still post it.
The returned error carries no `way forward:` clause, unlike the three refusals beside it.

### [LOW] The relay-down warning overpromises

**Location:** internal/creelengine/verbs.go#relayDownWarning

**Issue:** The warning says the message "waits in its mail until the relay starts".
The relay's outbound loop counts mail older than `staleMailAge` (24 hours, `slackengine/outbound.go:30`) as stale and never posts it.

### [LOW] EXTERNAL is unpinned

**Location:** internal/creelengine/machinemail_test.go#TestStore_PostToTheSlackRelay

**Issue:** EXTERNAL is `KindExternal` (`seam.go`, `ExternalSender`), so `slackRelayCheck` asks the bridge, which refuses it as no orch.
`creelengine/doc.go` and the commit message name EXTERNAL, but neither new table has an EXTERNAL row.

### [LOW] The orch's Slack guidance is stale

**Location:** contracts/orchrole/role-slack.md:7

**Issue:** The file says "mail you post to `<shortname>:SLACK` is dropped" while the button is off.
After the diff, a new post is refused with a way forward, and a post while the relay is down warns.
This file is what the orch reads before it posts to Slack, and it does not prepare the orch for either answer.

### [LOW] The change description misplaces the turn hook

**Location:** _lym/landing/summary.md

**Issue:** The summary says lym's own program senders, "such as the dobby reminders and the slack turn hook", keep their behaviour.
The hook posts through `orchClient`, the orch strand's client (`slackcli/hook.go`), so it is a strand sender and passes through `RelayPostReady`.
Its own `ForwardingOn` check makes a refusal unlikely in practice, but a down relay now gives it a warning.

## Rejected

- Pair orch with forwarding on wrongly refused (lens:code-correctness finding 3): only the hub's orch and a beam's orch have a slack button to turn forwarding on, the rule `SendReady` already applies, so no reachable pair orch has forwarding on.
- Tests for every bridge read failure (assist:assist F4): those paths only propagate errors; the reachable defect, a refusal with no way forward, is F4 and carries one failing-callback row.
- The help-text removals in `fe73fb3e`: the 2000-byte `maxLongBytes` cap forced the cut, and `creelcli/doc.go` still documents idle wake, the reply-command mark and the resolved `--parent` receiver; no finding.
- Severity of the `delivered: false` finding at MEDIUM (assist:assist F2): kept as F3 at LOW, because the sender is told with the state and a way forward, which is the entry's stated purpose.
- Severity of the stale role file at MEDIUM (lens:code-affected-set): kept as F7 at LOW, because the refusal itself names the way forward, so an orch is not stranded.

## Review coverage

Range `5823c67e3bb8bc2e9aab7aed301ffcb08a3f424c..HEAD`, three commits, judged against the board entry; the decision record carries no added entries and no declared PATTERN changes.
Focus settled:
- The help removals hold (Rejected).
- `RelayPostReady` matches the relay's own drop rule for EXTERNAL, an orch thread and a beam's orch.
  The `TaskSlug` and `Format` failures return unwrapped (F4).
- The refusal precedes interrupt and store.
  The `FormatProgram` error cannot occur with a daemon's valid shortname.
  A receiver spelled differently is a different mailbox that the relay never reads.
- `ReceiverLive` and `Warning` reach the plain and the waiting response.
  The Reply path is F1.
- The shared `Refuse` map has no cross-effect on existing tests.

No gate was run in this review; the verify gate covers build and tests.
