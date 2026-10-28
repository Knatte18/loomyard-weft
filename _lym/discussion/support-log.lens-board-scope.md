```yaml
lens: board-scope
outcome: reported
```

Task: check the creel-agents decision record against the board (entry goal points, touching entries, peers, decision notes).

**Findings**

1. MEDIUM, class `decision`, in Open risks / Auto-mode assumptions. Is the operator's 2026-10-27 wait met? The entry says the task waits until the operator has seen creel's hook delivery reach a working agent mid-turn in a real run. Its latest section (creel-hook-post landed, 2026-10-28) still says that wait "stands". The record raises this correctly under Open risks. Keep it as a question for the parent; no fixer should drop it. Evidence: the board entry body's "Operator decision (2026-10-27)" and its closing line of "Hook delivery landed".

2. LOW, in Decisions D5, D13 and the support log's Rejected alternatives. The record never answers the operator's 2026-10-23 view that SendMessage works well between Claude sessions, with each engine using its own messaging. It drops SendMessage for Claude-to-Claude messages as well, and D13's scan makes that permanent. The support log only rejects the relay-agent half of that idea. The goal ("not SendMessage") supports the choice, and hook delivery removed the cost the operator objected to. Still, the record should state the trade-off once: Claude-to-Claude now also goes through creel, and hook delivery means no extra read turn. Evidence: "Operator view, not a decision (2026-10-23): SendMessage works well" and "Operator idea (2026-10-23): each engine's own messaging" in the entry.

3. LOW, in Scope / D8. The entry says `lym shuttle send` and `lym shuttle interrupt` "move behind creel", while D8 removes them. That reading is consistent with the entry, but the record should say that removal is how it reads "move behind creel" so a reviewer does not take it as out of scope. Evidence: the entry's Scope bullet 2.

4. LOW, in Scope. The creelengine "Unbuilt" section names a stage 3 item, "the parent-review gate posts to the parent itself". The entry's brief also lists "parent review". Only the support log says this item is already met (the Go driver's relay through creel). The decision record has no line on it, so the plan writer cannot tell whether it was covered or dropped. Evidence: `internal/creelengine/doc.go`'s Unbuilt section and the entry's brief "Parent asks, parent review, ...".

**Goal points of the entry**
- Drivers' stop reports post, not duplicating batten's notices: met (D4; the Go driver already posts).
- `lym shuttle send`/`interrupt` behind creel: met by removal (D8; see finding 3).
- Parent directive asks through creel: met (D2).
- Hold notice names creel: met (D3).
- Other SendMessage stencils move to creel, Claude-only declared through `Spec.Requires`: met (D5, D7, D13).
- Risk that the hold notice and the directive reach different sessions: met (D1 and D3 use the same resolver).
- Reply discipline:
  - `read` shows the reply command: already landed.
  - Shared stencil rule: met through D2/D5 and shuttle's existing system-prompt sentence.
  - An inject never carries a question: met (D5, orch).
  - Unanswered messages journaled: met (D10, D11).
  - Expect-reply flag, with `--wait-reply` implying it: met (D9).
  - Bounded chase: met (D10).
  - Time limit as a config key (an orch recommendation, not a decision): replaced by a constant, with the reason stated. No contradiction.
  - Chase logging: met (D11).
  - Report per receiver role: met (D12).
  - Open question on the flag name and `defer`: settled in Auto-mode assumptions.
- Operator decision 2026-10-15 (stays a note until promoted): the task is now running, so promoted. No conflict.
- Operator decision 2026-10-27 wait: see finding 1.

**Entries, peers and parent commits checked**
- `codex-engine` (E2): none. The record leaves the Codex engine and Codex hook delivery out of scope. E2's step 3 (engine-keyed include sets) does not conflict, since D5 makes the message sites engine-neutral instead of keyed per engine.
- `claude-md-lym-rules-out` (T3, dependency): done. None.
- `test-skips-os-tags` (peer): text-only overlap on `internal/creelengine/server_test.go`. None.
- `board-readme-per-project` (peer): none.
- `codex-code-review-bench` (peer): none.
- Parent commits: `lym loom peers` lists none.
- Note `creel-carry-agent-mail`: not on the board list, so not reachable. The record lists it as out of scope, with no conflict.

**Decision notes**
- No `Decision for creel-agents:` note is on the board list, so none was read and none was out of reach.
- The record has no `Added after Discussion` entries.

**Tool calls**
- `lym board get codex-engine`: my context had only that entry's one-line brief, and I needed its body to check for an ownership or design conflict.
