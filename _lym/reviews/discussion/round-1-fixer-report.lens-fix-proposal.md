```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I tried to break each change the fixer made exactly as the reviewer proposed it. Two MEDIUM and two LOW findings.

**1. MEDIUM: F6's give-up after 5 tries drops the whole mailbox during a network outage**
- Changed text: D5, "After 5 failed tries of one message with an error other than a `429`, it posts one line naming the message's id and sender and counts the message as handled, whether that line posts or not."
- Case: the laptop sleeps, the VPN drops or DNS fails. Every post then fails with a transport error, which is "an error other than a `429`".
- D4's backoff (5, 10, 20, 40, 60 s) uses up 5 tries in about 2 minutes. Each queued message is then "handled". The notice line also fails, and "whether that line posts or not" still counts the message as handled.
- So a short outage silently loses every outbound reply and reminder. Those are what the at-least-once bound in D5 is meant to protect.
- The cap should count only Slack's own refusal of the message (an `ok: false` answer such as `msg_too_long`), not a failed transport or a 5xx.

**2. MEDIUM: F8's new D6 bound is false, and the F1 fix shows how to break it**
- Changed text: D6 Bound, "presence is changed only by the operator, an orch, a process outside every pane, and the relay …; no other agent strand can move the operator's messages and reminders between the terminal and Slack."
- Case: a non-orch strand double-forks a `lym slack back`, the same thing D3 bullet 2 now tells the relay starter to do. The process leaves the pane tree, the pane-ancestry check finds no pane, and the command runs.
- D7's Bound already admits this for creel: "any process of the operator's user outside a pane can post as `SLACK`".
- The first clause matches the reviewer's proposed bound. The second clause, "no other agent strand can …", is the fixer's own addition and overclaims. Keep only the first clause, or say that a strand which deliberately leaves its pane is not stopped.

**3. LOW: F13's angle-bracket rule doesn't say which forms are URLs**
- Changed text: D4, "turns `&lt;url|label&gt;` and `&lt;url&gt;` into the URL, and leaves the rest … as Slack delivers it."
- Case: a channel mention `&lt;#C123|general&gt;`, a broadcast `&lt;!here&gt;` or a user-group mention `&lt;!subteam^S1|@team&gt;` has the same shape as `&lt;url|label&gt;` or `&lt;url&gt;`.
- Read literally, the rule turns `&lt;#C123|general&gt;` into `#C123` and strips the brackets from `&lt;!here&gt;`.
- The rule should say it applies only to a bracketed value starting with a URL scheme (`http://`, `https://`, `mailto:`), and that every other form stays as Slack delivers it.

**4. LOW: F1's "never in a pane's process tree" bound assumes no subreaper in the pane tree**
- Changed text: D3 bullet 2, "the relay is reparented out of the starter's process tree … Bound: the relay is never in a pane's process tree at its first read".
- An orphan is reparented to its nearest subreaper ancestor, not always to init.
- lym sets no subreaper: a grep of `internal` and `cmd` for `SUBREAPER`/`Subreaper` found nothing.
- If a pane-side ancestor (the agent CLI or the tmux pane process) set one, the relay would end up under it and stay in the pane tree. Then "waits until its parent is no longer that intermediate" would pass while the bound fails.
- I can't tell from this repo whether any pane process sets a subreaper. The bound could say "reparented to init or the user's systemd subreaper", or the relay could check that its new parent is in no pane.

I checked the prime-only scope of F3 and found it correct. `EndDobbies` ends processes under `dobbyOwner(worktree)`, and that is `runproc.HubOwner` only for the prime (`internal/fabriccli/anchordeps.go:85-90`), so only a prime anchor change ends the relay.

I found no breakage in the proposals for F2, F4, F5, F7, F9, F10, F11, F12, F14, F15 or F16.

Further tool calls:
- grep for the `EndDobbies` wiring and for subreaper use in `internal` and `cmd`: needed to test F3's scope and F1's bound, and my context lacked both.
- `sed` of `internal/fabriccli/anchordeps.go:240-280`: needed the body of `EndDobbies` to see which owner it ends, which my context lacked.
- grep for `dobbyOwner`, `DobbyOwner`, `OwnerOf` and `HubOwner`: needed to find how a worktree maps to its owner, which my context lacked.
- Read of `internal/fabriccli/anchordeps.go:80-95`: needed the body of `dobbyOwner`, to confirm only the prime maps to `HubOwner`.
