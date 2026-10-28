```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each bound, guard, ordering promise and code claim in the decision record.

**Findings**

1. **MEDIUM. D9 / Constraints: the sentence's `"` breaks an existing init guard.**
   - D9 puts the sentence on `--append-system-prompt` "after the deny notice".
   - `claudeengine/settings.go` has `noticeTextForbiddenChars = steerTextForbiddenChars + "\n\r"`, where `steerTextForbiddenChars` is `'"\`. `init` panics when a notice contains any of them, because "the assembled notice rides one command-line argument that reed types with tmux send-keys".
   - The entry's verbatim sentence contains `"<text>"`. If it joins the deny notice under that guard, `init` panics; if it skips the guard, the reason the guard exists is broken.
   - The record has to settle it: lift the guard and say why, or change the sentence's quoting with the parent's leave, since the entry gives the text verbatim.

2. **MEDIUM. D2: the merged command has one stdin, and two readers need it.**
   - The claim "when it prints nothing or fails, the command records the turn end as today" fails as written.
   - D3 makes `hook-read` read the hook's standard input to find `agent_id`. The recording half (`recordingCommand`) also writes the payload from standard input into `events.jsonl`.
   - In one sh command, whichever runs first takes the stream. If `hook-read` reads it, the `Stop` record holds an empty payload. `ParseEvents` then loses `last_assistant_message` and `background_tasks`, so EventWaiting becomes EventStop, and the context reading loses `transcript_path`.
   - D2 should state that the payload is buffered once and fed to both halves.

3. **MEDIUM. D4 × D5: a ring can be followed by a typed bell.**
   - After a ring, the waiter exits 2 and its connection closes, so the daemon counts no live waiter.
   - Claude takes about 0.3 s or more to start the woken turn (spike case 3). The worker's next probe, 1 s later, can still read the pane as idle with claimable mail and no live waiter. D5 then types the bell into a session that is already waking.
   - Result: an extra turn, and a typed line during a turn start.
   - The record should say a ring counts as that mail set's bell, or that D5 waits a grace period after a waiter ends.

4. **MEDIUM. D7: "a refused interrupt stores nothing" covers one order only.**
   - The reverse case is open. Example: `post --interrupt r` with a 70 KiB body, or a receiver name that does not exist.
   - The interrupt is sent, the post is then refused, and the turn is lost with no message.
   - D7 should require the post to be validated before the key is sent.

5. **MEDIUM. D4 / D5: daemon restarts are not covered.**
   - A replaced binary ends the daemon, and that happens on every deploy, `deploy-dev` included.
   - The record does not say whether `hook-wait` reconnects; the standard waiting client reconnects up to MaxReconnects times.
   - If it does not reconnect, every idle strand loses its waiter on each deploy, and D5's typed bell becomes the normal path.
   - If it does reconnect, the new daemon must treat the reconnected waiter as live, and the waiter must not exit 2 on a dropped connection.
   - D4 should state which.

6. **LOW / MEDIUM. D2 bound: "the mail left then is reached by D4 or D5" is not proven.**
   - The 2.1.312 bundle's cap code counts "A hook blocked the turn from ending … consecutive times", and the rewake enqueue sets `stopHookActive:true`.
   - Whether a rewake counts toward the cap, or is suppressed by it, is not shown.
   - Case: an orch answering batten notices in text with no tool call chains rewake, block, rewake, block. Once the cap fires with a live waiter, D5 stays silent, because it needs no live waiter. If rewakes are suppressed too, the mail is stuck until the operator types.
   - The record should fall back to D5 when a ring produces no turn, or lower the claim to an open risk.

7. **LOW. D5: a waiter left from a killed session can block the bell.**
   - Case: the tackler re-spawns a row and its claude is killed with SIGKILL.
   - The `hook-wait` child is orphaned and stays connected. Claude enforced its timeout and is dead, so nothing ends it before a daemon restart.
   - The new session, interrupted before its first `Stop`, has a "live waiter" that rings a dead parent. That repairs itself after one ring (the connection closes), but the liveness rule is weaker than "connection open".

8. **LOW. D1 / acceptance "8 KiB budget … in full": hook output size is unverified.**
   - 3422ceac left "passes 8 KiB of additional context unshortened" as a check to do by hand.
   - D1 now sends the lines twice (`reason` and `systemMessage`), roughly 16 KiB plus JSON escaping.
   - The bundle has output spill logic (`skipSpill`, a length threshold). If a large hook output is spilled to a file preview, "in full" breaks.
   - Either the fact is researched now or the budget is set with a margin.

9. **LOW. D4 bound: "a stale bell … delivers nothing" may be wrong about where the bell lands.**
   - The rewake enqueue uses `priority:"next"`, which may join the running turn as a queued attachment rather than the next turn.
   - It is still harmless, but the stated bound may be wrong.

**Claims attacked**

| Claim | Result |
|---|---|
| D1: claim, budget and acknowledge order unchanged | held; size claim weak (finding 8) |
| D2: cannot loop | held: each block claims at least one message whole |
| D2: records the turn end as today on nothing or failure | broken (finding 2) |
| D2: cap bound reached by D4 or D5 | not proven (finding 6) |
| D3: test `agent_id` only, `agent_type` is set by `--agent` | held: the bundle's input builder sets `agent_type` from a default, `agent_id` only from an agent context |
| D3: the cost bound | held |
| D4: newer waiter ends the older one | held |
| D4: live while the connection is open | weak: orphan waiters (finding 7), daemon restart (finding 5) |
| D4: draft does not block | held, per spike case 3 |
| D4: stale-bell bound | weak (finding 9) |
| D4: 86400 s under the JavaScript timer ceiling | held |
| D5: once per new set | broken against D4 rings (finding 3) |
| D5: shuttle-launched strands only | held as stated; the mechanism is left to plan |
| D5: pause and hold | held |
| D6: agent mail in full, reminders gone | held; no `codexengine` exists |
| D6: old journal records still fold | held, per `PATTERN-tolerant-records` |
| D7: sender rules are the interrupt's | held |
| D7: a refused interrupt stores nothing | held; the reverse order is broken (finding 4) |
| D7: no running turn still posts | held |
| D8: quoting | held: no Go parser reads `MessageLine` outside creel; stencils match on the program name, not the quote shape |
| D9: launch and resume lines carry the sentence | broken by the char guard (finding 1) |
| Constraint: `Stop` does not run on a user interrupt | held (docs) |
| Constraint: rewake summary not dynamic for settings hooks | held (bundle) |

**Tool calls**

- Read `settings.go`'s steer and notice forbidden-character guard, and `orchengine` for shuttle use: my context did not hold the guard's definition.
- Grepped `creelcli` for the read verb's rendering: my context did not show `read`'s output format.
- Read `cli.go`'s read command around line 541: to see whether `read` output is text or JSON for D8 (it is text through `printMessages`).
- Dumped the 2.1.312 bundle around `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`: my context did not show how the continuation cap counts rewakes.
