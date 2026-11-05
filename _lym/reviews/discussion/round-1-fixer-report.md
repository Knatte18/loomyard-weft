# Fixer report — discussion round 1

Every finding was fixed in `_lym/discussion/decision-record.md`. The support log's stale Turn 3 lines for F1, F7 and F15 were updated to match.

## Changes

- F1, fixed as proposed: D3 bullet 2 now starts the relay through a double fork, and the relay waits until its parent is no longer the intermediate before its first creel read. It also states the bound that the relay is never in a pane's tree at its first read. The D3 criterion now has `lym slack start`, run from a non-orch strand's pane, bring up a relay whose first read creel accepts as `SLACK`. D6 bullet 2's wait is unchanged.
- F2, fixed as proposed: in D4 bullet 1, a first start writes the cursor as the newest message's `ts`, or the current time, before its first poll. The D4 restart criterion gains the case of a relay that never handled a message.
- F3, fixed as proposed (the restart option): D3 restarts the relay wherever a prime anchor change starts its dobbies again. The criterion requires the relay to be live after a prime anchor change with presence `terminal`. I chose restart over sparing the role because the anchor change moves the worktree path (`anchor.go` reopens logs under `fabricengine.WorktreePath(location, slug)`), so a spared relay would keep stale paths.
- F4, fixed as proposed (recommended option): `lym slack stop` sets `terminal` first, with D6's notices, then ends the relay. A bound states when it comes back, and the D3 criterion covers stop with presence `slack`.
- F5, fixed as proposed: the retention clause is gone from D5's Bound, now with the `retain` evidence. Mail stamped more than 24 hours before a read is skipped, and one line names the count and `lym creel log --to <shortname>:SLACK`. A new D5 criterion covers this.
- F6, fixed as proposed, then narrowed in the self-check: a persisted record under `.lym/slack/` holds the ids already posted or skipped. A message Slack refuses 5 times is replaced by one line. D4 resets the wait only after a post that succeeds. The D5 criterion now has a failure on the second message, then success, post each message exactly once.
- F7, fixed as proposed: D8's wait start is the time dobby first reads the child running at the row with no pause covering it. It is kept in dobby's `.lym/beam/` state record and cleared when the child leaves the row, stops running or is paused. The Bound line and the criterion now match, and the criterion gains the paused-and-resumed case.
- F8, fixed as proposed: `away`, `back` and `stop` refuse from a non-orch strand's pane, each naming its way forward. D6 gains a Bound line and a criterion. `stop` joins the list because F4 made it change presence too.
- F9, fixed as proposed: Constraints now say the limit is shared by every hub on the one app and that four hubs fit at 48 requests a minute. In D4, a `429` doubles that relay's floor, up to 60 seconds.
- F10, fixed as proposed: the D4 hold bound is now "for as long as the holder keeps renewing it, at most an hour past its last renewal".
- F11, fixed as proposed (bound option): D4's Bound says that a message routed to a beam that is torn down stays unread, and that `lym creel log --to <shortname>:<slug>:orch` shows it.
- F12, fixed as proposed: D5 adds the `@<slug>` hint to every sender that carries a beam slug, and the D5 criterion matches.
- F13, fixed as proposed, then narrowed in the self-check: Constraints gain the `conversations.history` paging, `oldest` and `Retry-After` facts and the angle-bracket forms, with their sources. In D4, bullet 1 pages until `has_more` is false. Bullet 5 turns URL forms into the URL and leaves other forms as Slack delivers them. A new D4 criterion covers this.
- F14, fixed as proposed: the D5 Rationale is narrowed to agents' and programs' outbound mail, and the relay's own answers are named as direct. The D6 criterion is split into CLI verbs and Slack texts.
- F15, fixed as proposed: D3, its criterion and D8's Rationale now say "tick" instead of "pass".
- F16, fixed as proposed: the third Open risks bullet is deleted.

## Self-check

- fix-consistency MEDIUM (the D3 criterion has `away` succeed from a non-orch pane, which D6 refuses): fixed as proposed. The criterion now names `lym slack start` only.
- fix-consistency LOW (D8 bullet 1 checks only while `slack`, but bullet 2 records whatever the presence): fixed as proposed. Bullet 1 now checks whatever the presence and reminds only while `slack`.
- fix-consistency LOW (D5 acknowledges single messages, but creel acknowledges a whole read): fixed as proposed. Old messages are now skipped, recorded, and covered by the read's acknowledgement.
- fix-way-forward MEDIUM (`stop`'s refusal names Slack texts that cannot stop the relay): fixed as proposed. `stop` now names the operator's terminal or an orch's pane.
- fix-way-forward LOW (an anchor change's failure path leaves the relay down): fixed as proposed. The relay restarts after a change that lands and after one that fails, and the criterion matches.
- fix-way-forward LOW (the give-up line names no way to read the message): fixed as proposed. The line names `lym creel log --to <shortname>:SLACK`.
- fix-claims MEDIUM and fix-proposal MEDIUM, merged (the D6 Bound overclaims, since a strand can leave its pane): fixed as proposed. The bound now covers only processes inside a non-orch pane and names the escape, as D7 does.
- fix-claims LOW and fix-proposal LOW, merged (the double-fork bound assumes no subreaper in a pane's tree): fixed by stating that premise in the D3 bound.
- fix-proposal MEDIUM (F6's give-up cap drops the whole mailbox during a network outage): fixed as proposed. Only Slack's `ok: false` answers other than `ratelimited` count toward the cap. A transport failure, a `429` or a `5xx` is retried, and the D5 criterion matches.
- fix-proposal LOW (the URL rule also matches channel and broadcast forms): fixed as proposed. Only `http://`, `https://` and `mailto:` values are turned into the URL, and the D4 criterion matches.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
