```yaml
lens: board-scope
outcome: reported
```

Task: a board-scope review of the codex-engine decision record and support log.

**Findings**

1. **MEDIUM, D2 (consistency).** The record leaves out part of the setup the operator chose.
   - The entry's Plan of 2026-10-27 names "the forked Astra fan at medium effort with block v2" as the setup. The goal of 2026-11-03 builds on the benches it ran.
   - The 2026-10-28 yardstick report adds a prompt bullet that names the decision record's decisions and acceptance criteria. Without it, the fan missed every divergence from the record.
   - D2 gives the assistant only the chair's own instruction files. It never mentions the block v2 reading directions or the yardstick bullet, so the assistant runs a setup no bench measured.
   - Evidence: the entry body's "Plan (operator, 2026-10-27)" and "Review bench" sections, and the `codex-review-yardstick-2026-10-28` paragraph.
   - Fix: carry block v2 and the yardstick bullet into the assistant's prompt, as an engine block or an assistant-only part. Or state in D2 why the chair's files already cover them.

2. **MEDIUM, D2 against the entry's Step 4 (decision).** Is a fan of Codex subagents in scope while the entry defers forks?
   - The entry defers "Forks and orch parity" (Step 4).
   - It also says "a `codexengine` that runs forks implements [`OutstandingForkReader`] as part of its fork work".
   - D2 has the assistant fan out over the round's lenses through Codex's own `spawn_agent` subagents. It has no fork audit and does not implement `OutstandingForkReader`, so it builds part of Step 4 without the machinery the entry ties to it.
   - Question for the parent: should the assistant fan out through Codex's in-turn subagents now, with no lym fork audit? Or should it run solo until Step 4?

3. **LOW, D2 and D3 against the entry's "First candidate roles" (scope).** The entry plans a fallback: "If a twill fan on Codex works poorly, the review runs as strong single Codex agents instead."
   - D2 ties the assistant's fan to the chair's fan for the round, and D3 adds no fan key of its own.
   - So the operator cannot run the assistant solo in a round where Claude fans out, which is the fallback the entry names.
   - The support log rejects such a key as "no need shown", but the entry shows the need.

4. **LOW, D5 (consistency).** D5 says it "follows the operator's 2026-10-24 decisions", but it departs from one of them.
   - The entry says "Each write is a read-modify-write under an `internal/lock` file lock, written to a temp file and renamed into place." That describes the home's `config.toml`.
   - D5 instead rewrites `config.toml` whole from lym's own record, so Codex's own writes to that file are dropped at the next launch.
   - The choice may be sound, but D5 should name it as a departure rather than claim compliance.

**Goal points of the entry**

- Codex runs as an extra seat beside the Claude reviewer, the chair consolidates its report, and Codex never gates (goal of 2026-11-03): met, by D1 to D3.
- A real codexengine built from claudeengine's needs: met, by D4 to D10.
- Step 1 (the home, launch, hook events, startup classifier, send and paste, completion on `Stop`, interrupt on `turn_aborted`): met, by D5 to D8.
- Step 2, skills synced into the Codex home: not met as worded. D9 loads skills by path instead and raises this under Open risks.
- Step 2, deny guardrails as `PreToolUse` hooks: met, by D10.
- Step 3, engine-keyed include sets that Burler review's stencils use: met, by D11.
- Step 4, forks and orch parity, deferred: the record defers lym's fork machinery, but see finding 2.
- `-a never -s workspace-write`, with no reliance on saved approvals: met, by D6 and the Constraints.
- One shared lym Codex home with an `auth.json` link and a `sessions` link, the operator's `~/.codex` untouched: met, by D5 and D13.
- Trust: add the worktree root, drop gone folders, lock plus atomic write, pre-set `[tui]`, check after startup: met, with the departure in finding 4.
- Codex effort at most `high`: met, by D4.
- A Codex reviewer gets the same context as the Claude reviewer, PATTERN overview and agent instructions included: met, by D2 and D12, with the heddle-target risk raised.
- Model aliases in the `sol[medium]` form: met, by D4.
- Heddle `codex-prompt` read through the shuttle seam, and `codex-file` rendering `AGENTS.override.md` so the engine writes no instruction file: met, by D6 and D12.
- The block v2 setup and the yardstick bullet: not met, see finding 1.
- Codex runs capped and spent sparingly: met, by the one-run llm smoke test in the acceptance criteria.

**Entries, peers and parent commits checked**

- `lookup-agents-through-shuttle` (F15): no conflict. It is a later consumer of a shuttle-spawned Codex agent.
- `claude-account-weights` (F12): no conflict. It concerns the claude engine's launch environment only.
- `run-pins-its-version` (E3): no conflict.
- Peer `darn-writer-runs-no-gate`: only text overlap, on `contracts/stencils/stencils.go`. No conflict.
- Peers `batten-remote-order`, `webster-subpath-anchor`, `window-titles-show-handle` and `fabric-test-speed-survey`: no conflict.
- Parent commits: `lym loom peers` lists none.

**Decision notes**

The board shows no `Decision for codex-engine:` note, and the record has no `Added after Discussion` entries.

**Tool calls**

None. My context already held the board entry, the board list, the two touching notes and `lym loom peers`.
