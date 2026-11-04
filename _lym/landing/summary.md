# A shuttle session gets its prompt and its creel hooks at launch

Two launch faults made a Claude session started through `lym shuttle` miss what it needs: a darn writer that never got its task, and a `lym shuttle open` session that got its creel mail only as reminder lines.

## The deferred prompt is typed again when its turn never started

A launch that loads skills carries the skill-load message on its launch line and defers the prompt pointer to a typed line (`Launch.PromptLine`).
The trace of the reported Darn-Write run (task `transitional-report-note` on the `cs` hub) shows how it went wrong.
After the load turn ended, the display-only segment color waited out the whole `send_ready_timeout_s` on a session it read as not idle.
Meanwhile the operator typed into the pane.
The prompt send then failed its landing check,
and `loadSkillsThenPrompt` accepted *any* turn start past the pre-send offset as proof the prompt had landed late.
The operator's own turn start passed that check, so Start returned the run and Wait held turn ends of an agent that never had its task.
`Run.DeliverPendingPrompt` had the same blind spot: any turn start past the prompt offset counted as the prompt's, and with a zero offset the skill-load turn did too.

- `promptTurnStartedSince` (send.go) counts only a turn start whose typed instruction equals the prompt line.
  `DeliverPendingPrompt` uses it, holds the send lock for the whole call, and types the line at most once per handle.
- Wait calls `DeliverPendingPrompt` at a held turn end of a run with a deferred prompt line when no pause covers it and no later turn start was read (`deliverPromptAtHold`, hold.go).
  A held turn end it types the prompt at sends no hold notice; a failed delivery is logged at Warn and tried again at the next held turn end.
- A run `Attach` reconstructs is marked `attached`, and Wait first delivers its pending prompt when its output files are missing (`deliverPromptOnAttach`).
  This covers a driver that died after persisting the prompt offset but before typing the line, which leaves no later turn end to act on.
  burler's explicit call on its probed handle stays; the once-per-handle rule keeps Wait from typing a second time.
- The late-landing acceptance in `loadSkillsThenPrompt` is unchanged: it keeps a session the operator is talking to from being torn down, and the held-turn check now repairs the case it misjudges.

## A session opened with `lym shuttle open` in a hub is a full creel agent

`SessionOpener.OpenLaunch` now takes a directory of the session's own, `<anchor>/.lym/shuttle-open/<token>`, beside the exit marker of the same token.
`Runner.Open` mints the token before calling it, forwards `Launch.CreelHooks` onto the strand, and removes the directory when the strand add fails.
Claude's `OpenLaunch`, told a hub board directory, writes two files into that directory, names both on the fresh and the resume line, and marks the launch `CreelHooks`:

- `settings.json`, holding creel's hook delivery and nothing else: the PostToolUse mail hook, a Stop delivery and the idle waiter.
  The Stop delivery is the run's merged command without the recording, since an opened session has no events file, and it always exits 0 so an echo reaches the operator.
  Claude Code stacks `--settings` over the worktree's settings, so every other setting stays the worktree's.
- `append-system-prompt.txt`, holding the creel trust sentence alone.

Told no board directory, it writes nothing and marks nothing, as before.
The creel Stop command is split into `creelStopDelivery`, shared by the run's merged command and the open command, and the waiter entry into `creelWaitHookEntry`.
The flag-directory resolution Prepare did inline is now `creelFlagDirForHook`, which both launches use.

## Design decisions

- The fix lives in shuttleengine's Wait and Attach, not in a loom or seat caller, so every recipe whose run defers its prompt gets it, and so does every caller that attaches.
- The opened session also gets the creel trust sentence, not only the hooks: the hooks hand it `[creel]` lines, and the sentence is what tells a session to follow them.
- The color still precedes the prompt and can hold it for one send-ready window. It was the delay in the reported run, but with the held-turn redelivery it no longer loses the prompt.

## Bounds

- A prompt line the provider queued while a turn ran starts its turn only when that turn ends, so a held turn end read before then can type the line a second time.
- The prompt-turn match relies on the engine reporting a typed turn start's instruction verbatim; both registered engines do.

## Docs

The package docs of `shuttleengine` (Pending prompt, Opened sessions) and `claudeengine` (open launch, append-system-prompt file), `SessionOpener`'s contract, creelengine's list of pane-mail strands, and `docs/overview.md`'s shuttle entry are updated.
