---
round: 1
focus:
  - "internal/creelengine/receive.go and verbs.go: does the `deliver` path follow D3 in order (refuse checks, D1 resolve, duplicate/id-clash check before receiver rules, confirm with 10s timeout and no daemon start, store-lock repeat of the duplicate check) and never hold the store lock across a call to another daemon?"
  - "internal/creelengine/receive.go: on confirm `no`, is the message dropped to no agent, a drop event journaled, and the orch notice bounded per resolved sending hub (five notices an hour, fifth says more are journaled only, next hour counts skipped drops, map keyed on hub not claimed strand)?"
  - "internal/creelengine/outbound.go: does the worker meet D2 — oldest first per receiving hub, 30s call timeout, backoff 1s doubling to 1m, hour limit from the journaled post time across restarts, per-hub start cap of three that resets only on an answered call or restart, spawn logged and reaped, final `deliver` before journaling a return, and no return crossing an in-flight deliver?"
  - "internal/creelengine/outbound.go and the folded pool: is an outbound message held by no local strand (no mail flag, reminder, sweep, chase, settle, retention beyond pending), and does a delivered expects-reply message wait for its reply until retention?"
  - "Return notice reason text: is a receiver's answer cut on a rune boundary to 200 bytes, line breaks made spaces, quote marks and backslashes escaped, and quoted as data?"
  - "D5 trust: internal/creelconfig and its use at arrival — Guard category, local layer refused, loaded per arrival, load failure stores untrusted, trust only for exact `<sn>:orch` or `<sn>:<slug>:orch` on Linux and never `orch-N`."
  - "D6 stamps: do hook line, notice, unread reminder, `lym creel read` text and JSON, `--wait-reply` reply JSON and `lym creel log` render the stamp or fields as specified, with `from` kept bare in JSON, the 8 KiB cap measured on the stamped line with the widest int64 id, and pane-mail strands reminded not typed?"
  - "D4 replies: is a delivered reply accepted only when the answered id is an own outbound message that went to the replier's hub, are other ids refused, are waits ended and the original marked replied on both sides, and are forwards to a program or another hub, and settles toward the other hub, handled as specified?"
  - "D8 and D2 sender/receiver rules: are inject, interrupt, hold, release and `post --interrupt` to another hub refused naming `lym creel post`; are program/EXTERNAL senders and program receivers across hubs refused with their way forward; does `lym creel hubs` list live hubs with daemon-answering and trust columns?"
  - "internal/hubregistry and standalonestate: directory told not discovered, locked rewrite through internal/state, daemon records its own hub after the single-instance lock and prunes only entries whose path is gone or whose readable shortname differs, lookup refusals carry the way forward from D1; tests do not write the operator's record."
  - "Mechanical-rule reach: PATTERN-daemon-wakeups declared list in cmd/lym/daemonwakeups_test.go includes the outbound worker; PATTERN-cwd-resolution seam through internal/creelcli; PATTERN-agent-addressing, config-layers, daemon-wakeups and test-isolation edits land the declared text."
  - "D7/D9 agent instructions: the Claude trust sentence, parent directive and orch role carry the stamp wording and no longer trust another hub's orch strand; creelengine doc.go drops the 'later and not planned' line and documents D1 to D6; the board note `creel-cross-hub-trial` content matches D9."
  - "Test economy and the change description: new tests duplicating existing coverage, per-helper tests covered by public-surface tests, behaviors added without coverage, and any claim in _lym/landing/summary.md the diff does not bear out."
---

Seed focus for round 1 over the range `9d2029216daaf759c6888dd9b26959d51f2b5923..HEAD`.
Entries name where to look; none states a verdict.
