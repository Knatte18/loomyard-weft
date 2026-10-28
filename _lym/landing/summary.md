# Creel delivers Claude posts through hooks, visibly and in full

Creel now hands every post to a Claude strand through hooks that shuttle renders into the session's `settings.json`.
A scope change from the orch reaches a working agent at its next tool call, not only at its turn end, and no message body is typed into a pane for a Claude strand.
The operator sees each message in the pane, and the receiver acts on it.

Closes #871

## What changes

- Mid-turn: the `PostToolUse` hook runs `lym creel hook-read claude`, which prints `{"decision":"block","reason":…,"systemMessage":…}` instead of `additionalContext`.
  The operator sees the lines under the tool result, and Haiku acts on a `block` where it refused `additionalContext`.
- Turn end: the `Stop` recording command and the creel delivery are one command, so a blocked stop writes neither a stamp line nor a payload to `events.jsonl`.
  A blocked turn end therefore never reads as a finished turn to `ParseEvents`.
  Creel counts consecutive `Stop` blocks per strand and stops blocking at 6, under Claude Code's own cap of 8.
- Idle wake: a second `Stop` entry, an `asyncRewake` waiter (`lym creel hook-wait claude`), wakes an idle session with a fixed line and exit 2.
  The waiter claims nothing; the woken turn's `Stop` or first tool call delivers the mail as a visible block.
  The cost is one short model turn per idle wake, because a settings hook cannot show rewake text to the operator.
- Bell: when no waiter reaches an idle strand, the delivery worker types the same fixed line once per claimable set through the guarded pane send.
  Only a strand carrying the new reed field `CreelHooks` gets it, so hand-added strands, `lym shuttle open` sessions and sessions launched before the deploy get none.
- Main thread only: `hook-read` and `hook-wait` claim nothing when the hook input carries an `agent_id`, so a fork never swallows mail its context would lose.
- `lym creel post --interrupt <receiver>` validates the post, sends the guarded interrupt, then stores the message.
  It is refused before any key for a receiver without `CreelHooks`, and together with `--reply`.
- Sender names are quoted in `MessageLine` and in `lym creel read`: `[creel] #<id> from "<sender>": <body>`.
- Shuttle appends a trust and reply sentence to every Claude session it launches with a run directory.
  It goes through one `--append-system-prompt-file` flag beside the run's `settings.json`, replacing the `--append-system-prompt` argument the deny notice rode, because a native argument holding `"` is not carried reliably on Windows.
- `lym creel` help states the reply rule.
- Docs follow: the `doc.go` of `creelengine`, `creelcli`, `claudeengine`, `orchcli`, `docs/overview.md` and the orch role stencil.

## Design decisions to review

- A post is capped at the 8 KiB claim budget, replacing the 64 KiB `maxBodyBytes`, so every claim fits one hook field (10,000 characters, above which Claude Code shows a preview while creel has journaled the message read).
  A longer post is refused with the existing way forward: write the text to a file and mail its path.
  A reply to `EXTERNAL` keeps the 64 KiB cap.
- `lym loom decision add` refuses an entry whose own notice would pass the budget, before it records anything, and LOOM's notice sends one post per entry when the joined line would.
- A message an old binary stored whose line passes 10,000 characters is left out of every claim and stays for `lym creel read`.
- Strands without a hook-delivering engine keep today's pane path (notice, reminder, backoff, notice cap).
  That covers an empty `Provider` and any engine `ToolBoundaryRenderer` does not find, so a hand-added Codex session still gets its mail.
  Only a strand whose engine renders hook delivery loses the pane path.
- The `Delivery` seam's `Idleness` answer gains `NoTurn`, true when no turn runs whatever the input box holds.
  A woken turn records no `UserPromptSubmit`, so the daemon holds `NoTurn` false from a ring until a turn end recorded after it.
- Daemon state is in memory: a restarted daemon sets every strand's block count to the cap, so the chain's next `Stop` claims nothing and the ring delivers the rest, and it may ring or type the bell once more for a set it already woke.
- Sessions launched before the deploy keep their old settings and resume line, so they get no `Stop` delivery, waiter, bell or trust sentence until shuttle launches them afresh.
- Codex is out of scope; the engine-neutral seam stays and no Codex hook is rendered.

## To check by hand

- The real-Claude scenarios live in the LLM tier (`internal/creelcli/smoke_llm_test.go`) and were not run in the verify gates.
  Run them against a live Claude, covering a mid-turn block, a `Stop` block, an idle wake, three successive wakes, a daemon restart mid-chain, and a real fork.
- Open carry-over, the `journal-close-test-row-panics` finding: a journal-close test row can panic.
  Check the creel journal tests for it.
- Open carry-over, the `llm-smoke-bell-pane-assertion` finding: the smoke test's assertion on the bell in the pane may not pin what the scenario claims.
  Check it in `internal/creelcli/smoke_llm_test.go`.
- Open design points, answered only by the decisions above:
  - an oversized first message overflowing a hook field, handled by the line cap;
  - the continuation count climbing when resets are unordered, handled by applying a reset before the cap check;
  - the daemon restart losing the count and the woken marks;
  - the idle reading missing from the delivery seam, handled by `NoTurn`;
  - the bell misdirecting pre-deploy sessions, handled by the `CreelHooks` mark.

  Confirm each holds in `internal/creelengine/verbs.go` and `deliver.go`.
- Whether one extra model turn per idle wake is acceptable, and whether to keep the typed bell, are the operator's call.
