# creel stage 3: agents message through creel, not SendMessage

Every agent now reaches its parent, its seat peers and the orch through `lym creel`, never through Claude Code's `SendMessage`, so a non-Claude engine can take a role that has a parent.
A message that expects an answer is tracked: creel chases a receiver that read it and did not reply, records the outcome, and `lym report replies` shows which roles need chasing.

## What changes

- `lym creel post --parent <text>` resolves the asking agent's parent orch at post time and names it in the success envelope.
  The walk is batten's run-notice rule, now in the new `internal/orchroute` (moved out of `orchcli` so `creelcli` can reach it): from a pair, the nearest live orch from the pair's parent worktree; from a beam or the prime, from that worktree itself.
  It is refused, naming `lym reed list`, when the resolved worktree records no orch strand, and before any call when given a receiver argument or `--reply`.
- The hold notice names `lym creel post --reply <id>` and `lym creel post <strand>`.
  The burler, loom and webster runners' notifier wirings resolve their orch with the same rule `--parent` uses, so a held agent's question and its hold notice reach the same session.
- The parent directive, the LLM driver's notify rules, the darn writer, webster Merriam, the seat chair and advisor, the discussion advisor and the orch role message through creel; every question carries `--expect-reply`.
  Merriam's message to a fork by agent id stays, since it is in-session addressing under `ForkSubagents`.
- The parent directive trusts the parent, any orch strand of the hub, and mail from a lym program (an upper-case sender such as `<shortname>:LOOM`).
  The system-prompt sentence `claudeengine/settings.go` appends to every Claude session defers other mail to the role's own rules, so an advisor's answer stays advice for the chair.
- `RequirementPeerMessaging` is removed from the closed set, the Claude engine's capabilities and the seat specs.
- `lym shuttle send` and `lym shuttle interrupt` are removed.
  `lym creel inject`, `lym creel interrupt` and `lym creel post --interrupt` replace them, addressed by strand name; every refusal and doc that named the old verbs names the creel verb.
  `Runner.Send` and `Runner.Interrupt` stay for shuttle's own callers and creel's delivery.
- `lym creel post --expect-reply` marks a message as expecting a reply without blocking; `--wait-reply` implies the mark.
  The mark shows in `lym creel read`, the hook-delivered line and `lym creel log`, and a reply by the holder clears it.
- A read, marked, unanswered message is chased twice, one 15-minute interval apart, by a content-free creel post signed `<shortname>:CREEL`.
  One interval after the second chase it is `unanswered`: creel journals it, notifies the sender, and a waiting `--wait-reply` exits 1 with an `unanswered` outcome.
  The journal records `chased`, `chase-aged` and `unanswered` events, and `lym creel log --id <id>` shows the whole path.
- `lym report replies` counts expected replies, replies, chases and unanswered messages per receiver role over the journal's retention.
- A tier0 scan fails on `SendMessage` in stencils, prompt fixtures under `testdata/` and Go string literals, except the allowlisted fork-addressing line.
  `PATTERN-agent-messaging` records the rule; `creelengine/doc.go` replaces its "Unbuilt" section with the built design.

## Design decisions

- The chase clock counts only while the holder is live, unpaused and under no other sender's hold, and is rebuilt from the journal after a restart.
  The consumed time is journaled as `chase-aged` at the inject's persist interval; the first tick after a chaser starts charges, so the chase lands at a stated number of ticks.
- A chase is never forwardable.
  A reply to a chase answers the chased message and is accepted only from that message's current holder; the former-holder refusal is a fail-loud guard no test reaches.
- A holder that disappears after reading a marked message is found by the removed-holder sweep, whose subjects now include strands holding a read, marked, unanswered message; its two-miss rule settles the message `unanswered` with reason `holder-gone` and no chase goes out.
- Program trust is not limited to named programs.
  Lym verbs that post caller-supplied text as a program, such as `lym loom decision add`, therefore reach agents as followable mail, bounded by the 0700 lymtemp root and the operator's user, as creel's doc already states.
- Operators lose guid addressing with the removed shuttle verbs.
  An inject under a pause is held and typed after the resume rather than refused, and no `MessageTail` is appended unless the sender writes it.

## To check by hand

- The creel sender notice for an unanswered message is posted by `<shortname>:CREEL`, which makes creel a sender that others may answer; check in `internal/creelengine` that a reply addressed to it is refused or ignored with the way forward and does not loop.
- The real-Claude scenarios in `internal/creelcli/smoke_llm_test.go` and the LLM tier were not run; run them to see hook delivery of marked posts and chases in a real session.
- The chair–advisor channel now rides hook-delivered creel mail, which changes when a chair sees an advisor's answer; the first Discussion or Plan table with advisors is its first real run.
