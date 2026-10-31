```yaml
lens: board-scope
outcome: reported
```

Board-scope lens: checking the sampler decision record and support log against the board for contradictions.

**Goal points of `interactive-recipe`:**
- New loom recipe `sampler` beside tapestry and darn, under the loom contract: met (D1, D2).
- Batten works the same for sampler as for every other recipe: met (Scope Out, Auto-mode assumptions).
- The record is written mechanically from the board entry, as darn's is, and the agent appends what is settled with `lym loom decision add`: met (D4).
- The build segment is the interactive agent alone: met (D1, D5).
- Batten makes the worktree, opens the window and starts the agent with the body as its brief, and the agent waits for the operator: met (D5, D8). Opening the window still depends on `batten.open_window`, which is stated as an assumption.
- As many rounds as the operator wants, with the agent committing as it goes: met (D6 sets no timeout, D8 has it commit).
- The operator says done and the agent runs a `lym` verb that ends the build segment, like `pr-continue`: met (D6 `sampler-done`).
- The run then goes through the Code-Review perch (off by default), Describe, Finalize and Land: met (D1, D3).
- Result is a repository change landed as an ordinary commit: met.
- Result is a board entry written through `lym board`, landing empty: met (D8, last acceptance criterion).
- Result is a trial, where "the run can use a tmux server of its own": partly met (D9). The record gives the trial no server of its own and adds no mechanism. It sends a dev-lym trial to the machine's shared sandbox hub, and leaves tmux outside lym to the agent's `tmux -L` by stencil rule.
- The agent is not Discussion-Write and gets a short stencil of its own (stay in the worktree, commit, board writes through `lym board`): met (D8).
- It shares only the mechanism with Discussion-Write's interactive mode, which the entry names as `lym shuttle run --interactive`: met in substance. D5 uses PR-Review's unawaited session start instead. It is still an interactive shuttle session in a pane, so this is no contradiction.
- Dobby's quiet notice and shuttle's idle timeout are off for this recipe while the agent waits: met (D5 no deadline, D7).
- The recipe is named `sampler`: met.

**Findings**

1. MEDIUM, D9 / Goal, class `decision`. The entry says the trial run "can use a tmux server of its own". D9 provides no run-owned server and puts a lym trial in a sandbox hub that one machine shares across runs. Nothing in lym enforces the rule; only the stencil does.
   - Evidence: the entry body's third result bullet against D9 ("needs no new lym mechanism"), and Open risks ("two trials on one machine share it").
   - Question for the run's parent: does a shared sandbox hub plus a stencil rule satisfy "a tmux server of its own", or does the operator expect lym to provide a server per run?

2. LOW, D7 / Scope. D7 also changes the step watchdog's reading for `Beam-Await` and `PR-Review`. The board's waiting-on-operator point is only "for this recipe".
   - No board entry owns watchdog behaviour for those rows, so this conflicts with no other entry. It does go beyond the entry's ask, and the record discloses it under Open risks.

3. LOW, Out of scope / `pr-continue-wakes-row`. D6 gives the sampler wait an immediate file-event wake. The note asks for the same thing for PR-Review, and the record leaves PR-Review alone.
   - This is no conflict: the note's design is untouched, and Open risks names the divergence.

**Entries, peers and parent commits checked**
- `interactive-idle-park`: no conflict. It is out of scope and builds on D6.
- `pr-continue-wakes-row`: no conflict (see finding 3).
- `reed-status-buttons`: the trial case. Its fit is finding 1.
- `board-agent-button`: no conflict. That is a separate agent opened from a button, not a run.
- `seed-starts-at-start`: no conflict. Sampler's first row is Preflight.
- `bolt-dir-rename`: no conflict. It depends on this task and does not overlap.
- `board-viewer`: no conflict. It is an example result of a sampler run and claims no recipe work.
- `readme-rewrite`: the board has no such entry (`task: null`), so it is gone or done. Nothing to check.
- `pawl-loom-recipe` (done) and its landed commit `4f7d099d`: no conflict. The record keeps pawl's shared `loomrecipe` path and `decision add`'s beam rules.
- `lym loom peers`: no peers and no parent commits.

**Decision notes:** the board holds no `Decision for interactive-recipe:` note, and the record has no `Added after Discussion` entries. I could reach every note I looked for.

**Tool calls:**
- `lym board get` for `board-viewer` and `readme-rewrite`: these are touching entries named in the task body, and my context held only the list rows for them.
