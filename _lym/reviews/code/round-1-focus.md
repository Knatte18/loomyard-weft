---
round: 1
focus:
  - "internal/creelengine chase clock (D10): check that the clock counts only while the holder is live, unpaused and not under another sender's hold, that `chase-aged` is persisted at `agedPersistInterval`, and that a restart derives chase state from the journal without a third chase or counted downtime."
  - "internal/creelengine chase and reply paths (D10): check that a chase is content-free, that a forward of a chase is refused with the way forward, and that `post --reply` to a chase is accepted only from the chased message's current holder."
  - "internal/creelengine/sweep.go: check that `sweepSubjects` and the sweep's run condition widen to read, marked, unanswered holders, and that the two-miss rule settles each such message `unanswered` with reason `holder-gone` and answers a waiting `--wait-reply`, with no chase sent."
  - "`--parent` resolution (D1, D3): check that `lym creel post --parent` and all three runner notifier wirings (burlercli, loomcli, webstercli) resolve through the same `internal/orchroute` walk, and that loom's hand-back and landing notices keep their walk; check the refusals when the resolved worktree has no orch strand and when `--parent` is combined with a receiver or `--reply`."
  - "Removal of `lym shuttle send`/`interrupt` (D8) and `RequirementPeerMessaging` (D7): check that no refusal text, doc, test, sandbox doc or PATTERN file still names the removed verbs or requirement, and that `Runner.Send`/`Runner.Interrupt` remain for their callers."
  - "Stencil moves (D2, D4, D5) and the creel trust sentence (D6): check each changed stencil and `claudeengine/settings.go` against the decision record for dropped, weakened or contradicted rules, including the widened parent trust, the unchanged LOOM decision-notice sentence, and the seat advisor's mail staying advice."
  - "The `SendMessage` scan (D13): check that its scope covers stencils, `internal/**/testdata/` fixtures and non-test Go string literals, and that the sole allowlisted site is the webster fork-addressing line of a `ForkSubagents` role."
  - "`lym report replies` (D12) and journal events (D11): check the per-role counts against the journal's reply-to ids, marks, `chased` and `unanswered` events, and that the verb is read-only."
  - "Docs and PATTERN landing: check that `internal/creelengine/doc.go` replaces the unbuilt stage 3 section with the built design, that other module docs the change touches are updated, and that every declared PATTERN change lands."
  - "Test economy: check each new test for duplicated coverage and each added behavior for missing coverage, naming the covering or missing test."
---

Focus follows the decision record's decisions D1 to D13 and the 17 cards; the riskiest logic is the chase clock and sweep widening in creelengine.
