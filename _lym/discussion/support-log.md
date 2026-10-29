# Support log — creel-mail-delivered-once

## Interview

### Turn 1 — exploration (autonomous, no operator)

Read the board entry (T1) and the board; no `Decision for creel-mail-delivered-once:` note exists, and no other entry touches creel delivery (F4 `landing-watch` adds a creel notice, no overlap).
Read the landed commit `c49ba448`, which moved the Stop mail into additional context but kept `decision: block`, and the creel engine, creelcli and claudeengine docs and code for the claim, the ring and the settings.

### Turn 2 — upstream facts, from Claude Code 2.1.312's bundled source

The installed `claude` 2.1.312 binary's strings were searched for the hook rendering paths.

- Stop hooks: a blocking error yields a meta user message `Stop hook feedback:\n<reason>` and the per-hook `hook_blocking_error` attachment, rendered `Stop hook blocking error from command: "<command>": <reason>`; the reason also goes to the operator's `Stop hook error:` list.
  Additional context yields a `hook_additional_context` attachment, pushed into the same continuation list unless a mode check holds, and goes to the operator's `Stop hook feedback:` list.
  A non-empty continuation list continues the turn with `stopHookActive: true` and the block count raised against `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` (default 8).
- `systemMessage` renders to the operator as `<hookName> says: <content>`: the unexplained `Stop says:` line of the board entry.
- PostToolUse: a blocking error yields a `hook_blocking_error` attachment with the command; additional context yields `hook_additional_context`, which the terminal does not draw.
- asyncRewake exit 2: the model text is `<rewakeMessage ?? 'Stop hook blocking error from command "<hookName>":'> <stderr || stdout>`, the operator summary `rewakeSummary ?? "Stop hook feedback"`, enqueued as a notification with priority `next`; the stderr is escaped for `system-reminder` tags, not truncated.
  `rewakeMessage` and `rewakeSummary` are hook-entry keys in the settings schema, marked `@internal`.

These answer the board entry's two "Find" bullets and its first Open bullet.

### Turn 3 — design, auto-picked

Settled D1 to D5 as the decision record states; each pick is in the question ledger.

### Turn 4 — lens review

All six forks returned a report.

- existing-paths, BLOCKING, old waiter auto-acknowledges ring mail and shows only the bell: fixed, D3 claims only for a waiter whose request carries the takes-mail mark.
- interactions, bound-breaker and existing-paths (merged), MEDIUM, write succeeds but acknowledgement fails, or reconnect after the write, delivers twice: fixed, D3 exits the wake code only on a confirmed acknowledgement and never resumes after a ring carrying mail; the unchanged `hook-read` exception is now stated in Constraints.
- existing-paths, MEDIUM, an older daemon's ring with no lines: fixed, D3 exits 0 and the bell follows.
- bound-breaker, MEDIUM, D1 and D4 proven for 2.1.312 only: fixed, Open risks names the version dependency and the smoke rerun on an upgrade.
- board-scope, MEDIUM, logging covers hook claims only: fixed, D5 logs each bell send and Scope names the journal records that already log pane-path attempts.
- interactions, LOW, ring answered on hand-out, not acknowledgement: fixed, D3 answers the ring only on acknowledgement.
- walkthrough and bound-breaker (merged), LOW, the pause cannot be read inside the locked step: fixed, D3 reads it just before and bounds the window.
- bound-breaker, LOW, a second loss window when Claude Code ends the hook: fixed, folded into D3's bound.
- walkthrough, LOW, Claude Code rewrites marker patterns in the wake body: fixed, stated as a D3 bound.
- existing-paths, LOW, the `OutcomeRing` comment in `pool.go`: fixed, Scope names code comments and that file.
- bound-breaker, LOW, other versions may refuse the D4 keys: fixed, merged into the version risk; D4's bound now names old sessions instead.
- leanness, MEDIUM, D1's argument against the block: fixed, cut to one clause.
- leanness, LOW, D2's first sentence: fixed, cut to one clause.
- leanness, LOW, `c49ba448` constraint narrative: fixed, cut to the replacement statement.
- leanness, LOW, `PATTERN-daemon-wakeups` and `PATTERN-test-speed` bullets: fixed, cut.
- leanness, LOW, engines bullet duplicated by the seam constraint: fixed, cut from Scope.
- leanness, LOW, two auto-mode assumptions repeating Scope: fixed, cut.
- leanness, LOW, the operator-race risk: fixed, moved into D3.
- leanness, LOW, D3 and D4 overlap on rewake rendering: rejected, D3's sentence is evidence for its bound and D4's is its rationale.
- leanness and walkthrough, LOW, `[creel]` twice in the wake body: rejected, the prefix's `[creel]` makes the wake's first line match the trust sentence, and the repeat costs nothing.
- leanness, LOW, transcript criteria overlap the JSON criteria: rejected, the lens itself calls it no cut; one is unit, the other real-session.
- board-scope, LOW, `c49ba448` replaced on source evidence: rejected, no fix asked; the tier3 criteria stay.

## Rejected alternatives

- Keep `decision: block` on a Stop and only shorten or empty the reason: every block adds the command-text attachment and the operator error line, whatever the reason holds.
- Exit 2 with the mail on stderr from the Stop hook: also a blocking error, with the same command text and label.
- Keep the block on PostToolUse, as `c49ba448` did: it leaks the command text and the error label at every tool-boundary delivery.
- Keep `systemMessage` on a Stop: it doubles the operator copy beside the stop-hook summary's `Stop hook feedback:` line.
- Ring first, then a separate claim request from `hook-wait`: two round trips and a window where the claim comes back empty after the session was already chosen to wake; claiming in the ring's locked step is atomic.
- Acknowledge the ring's mail only at the woken turn's first Stop or tool call: needs a new correlation between the waiter and a later hook, and an unseen wake would still leave the mail claimed meanwhile.
- Record the delivery path as a new journal field shown by `lym creel log`: the board asks for logging of attempts, which the daemon log carries without a journal format change; a journal field records only successes.
- Claim for every ring whatever the waiter's binary: an older `hook-wait` acknowledges unasked and writes only the bell, so its mail would be journaled read and never shown.
- Resume the wait after a failed acknowledgement: the lines already on stderr would reach the model in the next wake while the released mail is handed out again.
- Exit 0 from `hook-wait` on any acknowledgement not confirmed: when the daemon journaled the read and only the confirmation was lost, the mail is lost as read with no copy shown; D3 exits the wake code once the acknowledgement is written and takes the visible duplicate instead, as `hook-read` does.
- Carry the ring's lines in a per-wake `rewakeSummary` printed on the hook's stdout: Claude Code 2.1.312 honors that key only for a hook of a plugin from Anthropic's own marketplaces, and the waiter is a settings hook.
- Carry the lines in the settings `rewakeSummary`: the label is fixed for the session when the settings are written.
- Shape the wake's summary as a monitor event, whose body the terminal does draw expanded: it leans on an internal notification shape the hook does not own, and the body is still hidden until the operator expands it.
- Echo the ring's mail to the operator through the bell or a pane notice: new typing into the pane, against `PATTERN-pane-text-injection`.
- Drop `[creel]` from `rewakeMessage`: the wake's first line would no longer open with the tag the trust sentence names.
- Rewrite the bell line: it is typed as a prompt whose turn ends at a Stop that D1 now delivers cleanly; nothing in the entry asks for it.

## Review rounds

### Round 1

APPROVED with seven findings, all fixed in the decision record; Q10 resolved.

- F1, MEDIUM, mode check deferred to the smoke run: fixed, D1 states the simple-mode guard and why no lym session reaches it, and the risk is cut.
- F2, MEDIUM, the Stop summary row clamps to 5 lines: fixed, D1 states the bound and judges it against the Goal, and its real-session criterion checks it.
- F3, LOW, the version risk names no trigger: fixed, the risk records the cross-version evidence, the self-update, the failure a breaking update causes, and the rerun as an operator action.
- F4, LOW, the ring wake's operator copy unstated: fixed, D4 states the notification's shape and the D3 criterion checks the operator copy; whether the body is drawn stays open as Q11.
- F5, LOW, the ring's answer path ends without releasing: fixed, D3 releases on every end before the daemon journals the acknowledgement, and D5 logs it as `released`.
- F6, LOW, a lost confirmation after the journaled read: fixed, D3's Bound names the second window and Constraints both.
- F7, NIT, only overstates how a ring is answered: fixed, reworded.

### Round 2

APPROVED with four findings, all fixed in the decision record; Q11 resolved, Q12 auto-picked.

- F1, MEDIUM, the ring wake's operator copy left to the smoke run: fixed, the 2.1.312 source settles that the terminal never draws the wake's body; D4 records the evidence, D6 gives the operator an echo at the woken turn's first claim, and the open risk is cut.
- F2, LOW, the ring's acknowledgement-failure policy unweighed: fixed, D3 exits the wake code once the acknowledgement is written and states the loss-against-duplicate choice; the exit-0 policy is a rejected alternative.
- F3, LOW, the client's resume path defeats never resuming: fixed, D3 keeps a ring outcome off the resume path, the Notes name `CallWatching`'s resume, and Scope names the doc sentence.
- F4, NIT, no outcome for a failed journal write after acknowledgement: fixed, D5 adds `ack-failed` at `Warn`.

### Round 3

APPROVED with one finding, fixed in the decision record.

- F1, LOW, D6 states no echo rule for a failed acknowledgement: fixed, D6 drops a handed-out echo at that acknowledgement and keeps a ring's echo for the ids before the failing write, journaled or skipped, D5's `ack-failed` lists the dropped echo's ids and none on a ring, and the D6 criterion checks both.

## Question ledger

- Q1 (resolved, research): why the harness shows a Stop block twice and labels it an error — every block adds a `hook_blocking_error` attachment with the command and an operator error line, beside the `Stop hook feedback:` message; Claude Code 2.1.312 source.
- Q2 (resolved, research): what writes `Stop says:` — the hook's `systemMessage`; Claude Code 2.1.312 source.
- Q3 (resolved, research): whether an asyncRewake output is shown to the model for sure — it is read after exit and enqueued with priority `next`; residual risk is a session dying in between, bounded in D3.
- Q4 (resolved, research): the 8 KiB claim budget and the rewake output — the rewake text is not truncated, and 8 KiB is under the 10,000-unit field cap.
- Q5 (auto-pick): Stop output — additional context alone, no block (D1); rationale in the record.
- Q6 (auto-pick): include the tool-boundary path (D2), since the goal covers every delivery and it leaks the same way.
- Q7 (auto-pick): the ring claims in its own locked step and acknowledges on write (D3), the board entry's own design.
- Q8 (auto-pick): set `rewakeMessage` and `rewakeSummary` on the waiter entry (D4), with the fallback bound.
- Q9 (auto-pick): "is logged" means a daemon log line per attempt (D5), not a journal field.
- Q10 (resolved, research): whether a lym session can hit the mode check that keeps Stop additional context from continuing the turn — no; the check is Claude Code's simple mode, `CLAUDE_CODE_SIMPLE` or `--bare`, the same in the installed 2.1.310, 2.1.311 and 2.1.312, and lym never passes `--bare` while reed's launch scrubs every `CLAUDE_CODE_`-prefixed variable.
- Q11 (resolved, research): whether the terminal draws a ring wake's body under its `creel mail` row — no; 2.1.312's task-notification row component draws the summary, status, duration and a background command's delivery note, and a body only for a monitor event, in neither the expanded nor the verbose view; a stdout `rewakeSummary` is honored only for a hook of an Anthropic-marketplace plugin.
- Q12 (auto-pick): the ring's operator copy is an echo the daemon keeps and the woken turn's first claim renders as `systemMessage` (D6), the first operator surface lym reaches without typing into the pane.
