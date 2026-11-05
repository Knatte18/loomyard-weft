```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens over the decision record. For each state the decisions add or change, I traced the code that already runs over it.

**Paths traced, per decision**

- D2, reply-wait output: `post --wait-reply` prints the reply, a returned message and an unanswered message through the shared `messageFields` (`internal/creelcli/cli.go:433-441`). It breaks D6/D7 (F1).
- D2/D4, reply size check: `Store.Reply` checks the line with `validateLine(sender.Name, text)` against the local unstamped form (`internal/creelengine/verbs.go:223-228`). It breaks D2's stamped-size rule for replies (F2).
- D4, reply after a chase: `Store.Reply` follows `ChaseOf` to the chased original before routing (`verbs.go:211-219`). Survives, as long as the cross-hub branch comes after that step.
- D4, settle notice: `noticeUnansweredLocked` sends a strand sender a notice only when the local listing holds it (`internal/creelengine/chase.go:246-254`). A sender from another hub is never listed, so it gets no notice. Survives D4.
- D4, chase body: `postChaseLocked` names `msg.From` and carries no sender-written text (`chase.go:203`). Survives: the body names the sender bare and carries no stamp, but it holds no text the sender wrote.
- D6, hook line: `MessageLine` (`internal/creelengine/reminder.go:34-41`). D6 changes it. Survives.
- D6, reminder: `composeReminder` (`reminder.go:66-79`). D6 changes it. Survives.
- D6, notice typed into the pane: `isNotice` types a message in full only when its sender is a program (`reminder.go:27-29`). A strand from another hub is reminded, as D6 says. Survives.
- D6, read output: `writeMessagesText` (`cli.go:590-618`) and the `read --envelope` JSON from `messageFields` (`cli.go:290-310`). D6 changes both. Survives.
- D6, inbox: `inboxCmd` lists each sender by bare name and unread count (`cli.go:706-709`). The record is silent. It shows no stamp but carries no body (F3).
- D6, log: `logEntryFields` (`cli.go:781-808`). D6 adds the sending hub's path and id. Survives.
- D7, instruction text: `creelTrustSentence` exists only in Claude's `settings.go:473`. The Codex engine has no creel trust sentence, so its sessions read trust only from the parent directive and the orch role, which D7 changes. The seat directives give no trust to an orch. Survives.
- D3, sender resolution: `serve` calls `resolve`, then `resolveSender`, before `dispatch` (`internal/creelengine/server.go:208-213`). D3 skips it for `deliver` and `confirm`. Survives, as stated.

**Findings**

**F1 — MEDIUM — design — D6 (and D7)**
- **Location:** `_lym/discussion/decision-record.md` D6, second bullet.
- **Summary:** The cross-hub reply that a `post --wait-reply` waiter receives has no trust stamp or trust fields in the record.
- **Detail:** The command returns the reply as `fields["reply"] = messageFields(*resp.Message)`, and a returned or unanswered message the same way. D6 names only `lym creel read`'s text and JSON as carrying the stamp and fields. The trial's own flow, `ly:orch` posting `--wait-reply` to `lyt:orch`, reads the answer through this output. The answer shows a bare `from` and no trust, yet D7 tells the agent to read the stamp.
- **Fix:** D6 names the `post --wait-reply` output, its `reply`, `returned` and `unanswered` fields, as carrying the trust and sending-hub fields.
- **Evidence:** `internal/creelcli/cli.go:433-441`; `internal/creelengine/server.go:417-420`.

**F2 — LOW — design — D4 (with D2)**
- **Location:** `_lym/discussion/decision-record.md` D4, first bullet; D2, size-check bullet.
- **Summary:** The size check for a cross-hub reply is not stated, and the existing reply path measures the unstamped local line.
- **Detail:** D2's stamped-size check is stated for the post. `Store.Reply` checks `validateLine(sender.Name, text)` against the local form. A reply near the cap passes at `--reply`, is refused by D3 step 1 at the receiver, and returns undelivered later instead of being refused at once.
- **Fix:** D4 says a reply to a message from another hub is measured by D2's stamped form.
- **Evidence:** `internal/creelengine/verbs.go:223-228`.

**F3 — NIT — consistency — D6**
- **Location:** `_lym/discussion/decision-record.md` D6.
- **Summary:** `lym creel inbox` lists a sender from another hub by bare name and no stamp, and the record does not say so.
- **Detail:** The inbox carries no body, so an agent reads no mail through it. Its unstamped sender still sits beside the stamped outputs.
- **Fix:** D6 states that the inbox lists the bare name, or gives it the stamp.
- **Evidence:** `internal/creelcli/cli.go:706-709`.

**Tool calls**

1. Searched the engine packages for the creel trust text, to learn whether any engine other than Claude carries one. The manifest held only Claude's `settings.go`.
2. Read `cli.go`'s `post --wait-reply`, inbox and log output, which are outputs that read message senders. The manifest had only `writeMessagesText` and `messageFields`.
3. Searched the Codex engine, the contracts and heddle for `[creel]` and the orch-trust text, for instruction sites beyond D7's three.
4. Read `Store.Reply`, the existing reply path D4 extends, which the manifest lacked.
5. Read the seat, Webster and Codex instruction lines found in call 3, to check whether any grants an orch trust.
6. Read the chase and unanswered-notice bodies in `chase.go`, the paths D4 touches, which the manifest lacked.
