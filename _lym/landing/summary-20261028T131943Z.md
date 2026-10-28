# Creel delivers every post to a Claude strand through hooks

Creel now hands a post to a Claude strand through hooks that shuttle renders into the session's `settings.json`, in full and visibly to the operator.
A scope change from the orch reaches a working agent at its next tool call, never only at its turn end.
No message body is typed into a pane for a Claude strand.

Closes #871

## What changes

- **Mid-turn:** `PostToolUse` runs `lym creel hook-read claude`, which prints `{"decision":"block","reason":…,"systemMessage":…}`.
  The operator sees the lines under the tool result, and the receiver acts on them at once.
  This replaces `additionalContext`, which the operator cannot see and Haiku refused.
- **Turn end:** the recording `Stop` command and the creel `Stop` delivery are one command.
  While claimable mail stands it blocks the stop and appends nothing to `events.jsonl`, so a blocked stop records no turn end.
  Creel counts consecutive blocks per strand and stops blocking at 6, under Claude Code's cap of 8, so no journaled-read mail is ever discarded by an override.
- **Idle wake:** a second `Stop` entry, `asyncRewake` with `lym creel hook-wait claude`, holds a waiting daemon call.
  The daemon rings it once per claimable set when the strand has claimable mail and no running turn (new `Idleness.NoTurn`).
  The waiter writes a fixed bell line to stderr and exits 2, and the woken turn's first tool call or `Stop` delivers the mail as a visible block.
  A settings hook cannot show rewake text to the operator, so the wake carries no mail.
- **Bell:** a strand marked `CreelHooks` that no waiter reaches gets the same fixed line typed once per claimable set through the guarded pane send.
  A hand-added strand, a `lym shuttle open` session and a session launched before the deploy have no mark and get none.
- **Main thread only:** `hook-read` and `hook-wait` claim nothing when the hook input carries a non-empty `agent_id`, so a fork or subagent never takes mail.
- **`post --interrupt`:** one daemon request that validates the post, sends the guarded interrupt with `interrupt`'s rules, then stores the message.
  A receiver without the `CreelHooks` mark is refused before any key.
  `--interrupt` with `--reply` is refused.
- **Quoted senders:** `MessageLine` and `lym creel read` render `[creel] #1 from "ly:orch": …`.
- **Trust sentence:** every Claude session launched with a run directory appends the trust and reply sentence through one `--append-system-prompt-file`, which also carries the deny notice.
  The text rides a file because a native argument on Windows does not carry `"` reliably.
- **Help and docs:** `lym creel` and `lym creel post` help carry the reply rule.
  Package docs, `docs/overview.md`, the orch role stencil and `internal/orchcli/doc.go` describe hook delivery instead of the reminder flow.

## Design decisions to know

- **Line cap replaces the 64 KiB body cap.**
  `post` refuses a message whose `MessageLine` passes the 8 KiB claim budget, with the existing way forward: write the text to a file and mail its path.
  A reply to `EXTERNAL`, which reaches only a waiting `post --wait-reply`, keeps the 64 KiB cap.
  A post between 8 KiB and 64 KiB, accepted before, is now refused.
  An old-binary message whose line passes 10,000 characters is never claimed and stays for `lym creel read`.
- **LOOM's decision notice** sends one notice per entry when its joined line would pass the budget, and `lym loom decision add` refuses an entry whose own notice would pass it.
- **Pane path kept where no hook engine exists.**
  A strand whose persisted provider is empty, or names an engine with no hook renderer, keeps the notice and reminder lines, so Codex and hand-added strands still get mail.
  Only strands whose engine renders creel hooks lose the pane path.
- **Stop count is in daemon memory.**
  A started daemon sets every count to the cap, so a restart mid-chain ends the chain at its next `Stop` and the ring delivers the rest.
  The woken-set marks are also in memory, so a restarted daemon may ring or type the bell once more for a set it already woke.
- **Cost:** one idle wake costs one short model turn before the mail turn.
- **Sessions launched before the deploy** keep their old settings and get posts once shuttle launches them afresh.
- New constants and timings (waiter timeout 86400 s, 30 s for an unanswered ring, 5 s grace before the bell, the Stop cap of 6) are the author's choices; the board entry fixes none.

## Check by hand

- Real-Claude behavior is covered by the LLM-tier smoke test in `internal/creelcli/smoke_llm_test.go`, which was not run here.
  Run it on a machine with Claude to confirm mid-turn delivery, the `Stop` block, the idle wake, the fork filter and `post --interrupt`.
- The journal-close test row in the `creelengine` delivery tests may panic; check that the row's setup closes the journal safely.
- The LLM smoke test's bell assertion on the pane text: check that it asserts the pane content it means to and is not flaky.
- `docs/research/creel-midturn-trust-2026-10-28/report.md` rests on Claude Code 2.1.312; re-check the hook facts on a newer version.
