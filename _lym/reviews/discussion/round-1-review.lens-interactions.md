```yaml
lens: interactions
outcome: reported
```

Interactions lens: check each pair of the record's decisions that act on shared state, and each criterion and constraint against the decisions it touches.

**Pairs checked:**
- D2 × D3 (which rounds ask × ask record keyed by generation and round): **held**. The ask record lives under `.lym`, so the run-dir archive in `archiveRunDir` (internal/shedadapters/archive.go:55-77) leaves it in place.
- D2 × D3, key flipped off between a round's ask and a restart of that round's review row: **broken** (F4).
- D2 × D5 (asking round × deadline cap): **broken** when the cap leaves no room (F3).
- D2 × D7 (Burler-Fix fixes parent findings × chair keeps them; Bouncer still approves): **held**.
- D2 × D7, next round's ask read from a parent answer whose finding the chair merged into a duplicate: **held**. D2 reads the parent's report, not the review.
- D3 × D4 (asks keyed by generation and round × an answer verb that takes only the slug): **broken** (F1).
- D3 × D5, restarted row keeps the first deadline while the restarted chair gets a fresh `review_timeout_min`: **held**, because the first deadline is the earlier one.
- D3 × D5, `no-parent` set at once and await answering ready on it: **held**.
- D4 × D5, one lock and the state leaves `open` once: **held**. A late answer before any await ran is accepted, which is harmless.
- D4 × D7, report copied after the table ends while the chair reads the `.lym` report before it ends: **held**.
- D4 × D6 × D7 (findings file written by the fork × chair reading the parent's severity and class): **broken** (F2).
- D6 × D2, fork's `decision add --by parent` mid-round while Burler-Fix edits the same record: **held** as recorded. The support log rejects this as existing behaviour, and `decisionVerb` refuses only while Discussion-Write runs (internal/loomcli/decision.go:893).
- D1 × D5, assistant cap and parent cap both measured from the table's start: **held**. The two waits overlap rather than add up, as long as D5's cap reserves room as `CapAssistTimeout` does.
- D1 × D7, both consolidation parts in one round: **held**, per the plan-writer note.
- Criteria and constraints against the decisions they touch: **held**, except the D4 refusal criterion (F5) and the D5 cap criterion (F3).

**Findings**

F1 — MEDIUM, design, D4 (decision-record.md:72, 80) against D3 (decision-record.md:62)
- A late answer can land on a newer ask about a different record.
- D3 keys asks by generation and round, but `lym loom parent-review answer <slug>` takes only the slug and writes to "the slug's current round".
- Example: generation 1's round-1 ask times out and the perch settles. An amendment then opens generation 2, whose round 1 asks again. The parent's fork, still answering the generation-1 mail, runs the verb and finds an open ask, so its findings about the old record are accepted against the new one.
- The bound at decision-record.md:85 ("only … the slug's current round") is exactly what allows this.
- Fix: the request line carries an ask id (generation and round), and the verb requires it and refuses a mismatch with a way forward.
- Evidence: decision-record.md:60-62, 72, 80; amendment.go:40-59 shows a new generation re-seeds round 1.

F2 — MEDIUM, design, D4 `--findings-file` (decision-record.md:77) against D7 (decision-record.md:121-122) and D6 (decision-record.md:108-112)
- D7 keeps "the parent's severity" and "the class the parent gives", but D4 defines the findings file only as "a markdown file of findings, each naming what to change".
- Neither D4 nor D6's orch-role section names where a finding's severity and class go, and the verb doesn't check either.
- So D7's mapping rules (`decision` or none becomes `design`, `BLOCKING` when no severity is given) have no defined input. With free prose, every parent finding comes out as `BLOCKING`/`design`, or the chair guesses.
- Fix: D4 names the file shape, for example one item per finding with optional `severity:` and `class:` fields. D6's section tells the fork to use it, and a D7 criterion reads it.
- Evidence: decision-record.md:77, 109-112, 121-122, 166.

F3 — MEDIUM, design, D5's cap (decision-record.md:91) against D2's asking rounds (decision-record.md:47-48) and the D5 criterion (decision-record.md:162)
- D5 caps the deadline "as `assist_reviewer_timeout_min` is capped". The existing cap, `CapAssistTimeout`, has a no-room branch that returns a zero bound and does not run the seat (internal/loomengine/review.go:69-75).
- D5 never says what an asking round does when the cap leaves no room. Under D2 such a round still sends the ask, with a deadline at or before the ask time, so the await sets `timeout` at once and the parent gets mail it can never answer.
- The D5 criterion covers only the case where the deadline is shortened, not the case where no time is left.
- Fix: D5 states the formula (chair timeout minus start allowance, chair keep-warm cap and a consolidation margin). It also says a round with no room sends no ask, logs a Warn naming both keys, and marks the ask with a reason the review records. A criterion covers that case.
- Evidence: review.go:64-84; decision-record.md:91, 104, 162.

F4 — LOW, consistency, D2's per-round key read (decision-record.md:40) against D3's restart rule (decision-record.md:64) and D7's "filled only in a round that asked" (decision-record.md:120)
- Round N asks with the key on. The operator then sets `discussion_review_ask_parent: false` and the review row restarts in round N.
- D2 says this round "asks nothing". D3 says the restart finds the ask record and keeps its deadline. The record doesn't say whether the restarted chair awaits and consolidates the still-open ask.
- If it doesn't, the ask stays `open` forever, and D4 keeps accepting answers that no chair reads.
- Fix: D2 states that the key decides only whether a new ask is sent, and that an ask already sent in the round is awaited and consolidated.
- Evidence: decision-record.md:40, 64, 80, 120.

F5 — LOW, consistency, D4 criterion (decision-record.md:159) against D4 (decision-record.md:80)
- D4 refuses an answer to an ask that is `done`, `timeout` or `no-parent`.
- The criterion tests only `done` and `timeout`, so the `no-parent` refusal has no reaching test, which `PATTERN-refusal-way-forward` requires (pattern/PATTERN-refusal-way-forward.md:9).
- Fix: add `no-parent` to the criterion.
- Evidence: decision-record.md:80, 159.

Tool calls: none. The inherited context held every file this lens needed.
