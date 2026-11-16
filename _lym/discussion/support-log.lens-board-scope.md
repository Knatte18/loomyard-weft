```yaml
lens: board-scope
outcome: reported
```

Board-scope review of the codex-same-as-claude decision record: fit with the board entry, the touching entries, the peers and the parent commits.

**Findings**

1. **MEDIUM — Acceptance criteria and D1: the parity test runs per role, but the entry asks for it per seat.** (class: scope)
   - The entry says: "for each seat a Codex model may run, render the launch for both engines and fail when the Codex one lacks a skill, passage or instruction the Claude one has."
   - The record's first criterion runs over "every role agents.yaml defines but orch" on a "fixture launch".
   - A seat's skills are more than its role's. The record's own evidence says the darn writer's seat carries the role's skills plus the hub's language skills. A per-role fixture therefore never checks the skills a seat adds.
   - The prompt content of each seat is checked only indirectly, through the D2 scan and the D4 prompt-engine refusal.
   - Fix: run the parity test over every seat in the seat tables, building each launch spec from the seat (skills, role, engine blocks). Or state why a per-role check covers every seat.

2. **MEDIUM — D11 leaves `codex-writer-bench-osl` contradicting itself.** (class: design)
   - D11 adds a section saying the entry's hand-built prompt steps are superseded by the bench rule.
   - The entry's body keeps its other bounds:
     - "The bench replays only the writer, never a whole run: no other producer runs, and nothing lands or is pushed"
     - "Exactly one Codex session"
     - "Check out the parent of `824eb2b6` into a scratch worktree"
   - The bench rule needs "the real producer with only the model swapped … in `run_settings`". A real darn run with `darn.writer` set runs the other producers and lands, and it runs from the current parent, not from the parent of `824eb2b6`.
   - Superseding steps 2–3 alone leaves a body nobody can follow.
   - Fix: D11 should either say which of those bounds the rule also replaces, or say that the bench's replay mechanics under the rule are for that entry's own run to settle.

3. **MEDIUM — D3 and D5 change the Codex assist seat in the middle of the `codex-assist-review-trial` (R1) trial.** (class: decision)
   - D3 replaces the thin assist copies of the edit directive with the full directive.
   - D5 moves a Codex seat's notices and passages from the prompt file into `developer_instructions`. That includes the assist reviewer seat.
   - R1's chosen runs, `prime-is-root-beam`, `hub-stop-button` and `machine-restart-shortcut`, are still drafts. Each carries `code_assist_reviewer_model: astra[medium]` with the `review-code` fan.
   - R1 gives its verdict once all three have landed.
   - If this task lands partway through, the three runs compare unlike seats.
   - Question for the run's parent: should R1's three assist runs all run before this lands, or all after it, so the trial's runs share one seat setup?

**Goal points of the entry**

| Goal point | Met? | Where |
|---|---|---|
| Claude-only stencil wording swept into engine blocks | met | D2, with the scan |
| Standing passages: the Codex set gives what the Claude set gives | met | D5, D7 (`agents-prompt`) |
| Instruction files: `CLAUDE.local.md` | met | D7, through the heddle `claude-file` → `agents-file` move; the record leaves implicit that heddle owns that file |
| Instruction files: the user's global `CLAUDE.md` | met | D8 |
| Parity test | partly met | per role, not per seat (finding 1) |
| Skills | met | D1 manifest skill loads, D8 listing |
| Guards | met | D1 manifest guard rows; the guard hooks reaching subagents is settled by research |
| Bench rule: the real producer, the model swapped, no hand-built prompt | met | D11, with the gap in finding 2 |
| Bench rule: the report opens with what each engine got | met | D10, D11 |
| The writer bench and its OSL twin rerun after landing | not scheduled by the record | that is the operator's call, so no finding |

**Entries, peers and parent commits checked**

- `codex-engine-trial` (project): no conflict. D11 adds to its Rules, which is allowed because it is another entry.
- `codex-support-levels` (R2): no conflict. It already says level 2 is judged after this fix and a rerun.
- `codex-assist-review-trial` (R1): see finding 3.
- `codex-writer-bench-osl`: see finding 2.
- Peer `weaver`: no conflict. Its record defers to this task: "A Codex spec may stand on a rung only as far as `codex-same-as-claude` has given Codex sessions what Claude sessions in the same role get". It also renders the edit directive per `PATTERN-stencil-ownership`, which D3 keeps.
- Peer `adopt-build-revision`: no conflict, since it touches no launches or stencils.
- Parent commits: `lym loom peers` lists none.

**Decision notes**: the board has no `Decision for codex-same-as-claude:` note, and the record has no `Added after Discussion` entries.

**Tool calls**

- `lym board get weaver`, and a listing of the weaver worktree's `_lym/discussion/`: the peer's body and the location of its record were not in my context.
- A grep of the weaver decision record for Codex, engine-block, edit-directive and heddle overlaps: the peer's record was not in my context.
