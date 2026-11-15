```yaml
lens: board-scope
outcome: reported
```

Task: review the decision record's fit with the board (board-scope lens).

**Findings**

1. **MEDIUM, class `decision`, D8 and D7: `lym adopt --release` lets the hub go on with failing tests.** The operator decided that nothing happens in the hub until adopt has finished and every failing test it found is fixed. The release escape clears the hold while the check still fails, so runs go on with failing tests standing. Question for the parent: does the operator want an escape from the adopt-fix hold at all? If so, is an operator-only `--release` acceptable? Evidence: the entry's "Operator decisions (2026-11-15)" bullet 2; D8's `--release` paragraph; question-ledger Q11 (auto-pick).

2. **MEDIUM, class `decision`, D8: the orch is up during the `adopt-fix` hold.** The record reads "no orch action" as "no run action" and starts the orch before the hold clears. The entry lists "no orch action" separately from "no run start" and "no dobby pass". Question for the parent: may the orch be up during the fix hold, if it can take no run action? Evidence: the entry's bullet "no run starts, no dobby pass, no orch action"; D8's step 7 and its Bound; Q10.

3. **LOW, class `scope`, D11: a kept pair is reported but never torn down.** The entry names the kept pair of the cancelled `claude-account-weights` run among pairs that "stand forever". D11 only lists kept pairs and posts a notice at each dobby start, so that pair still stands until the operator removes it. This meets the entry's "nothing tells the operator it stands" but not "nothing tears down a pair that a cancelled run left behind". If report-only is the intended answer, it should be stated as such against the entry. Evidence: the entry's "Left-over pairs stand forever" bullet; D11's second bullet; Q15.

**Goal points of the entry**
- One FSM per run, committed and pushed: met (D1, D2).
- Each transition pushes before it acts: met (D2).
- State git cannot hold is a cache: met (D3, D10).
- Adopt is pull, read and go on, with no reconcile step: met (D8). The catch-up and takeover read origin only.
- Adopt runs in the foreground: met (D8).
- Nothing happens until adopt is done and every failing test is fixed: partly met (findings 1 and 2).
- Adopt works from pushed state only: met (D8 Bound).
- Absence detection with an origin check first: met (D6).
- Adopt failures A1–A14: met (D2–D8, D11).
- Orch continues its session (A6, A7): met (D9, D10).
- Two owner spellings (A8): met (D5).
- Left-over pairs, including the workspace file and the `rerun` respawn: mostly met (D11; finding 3).
- Re-parent verb waits: respected (Scope Out).

**Entries, peers and parent commits checked**
- `adopt-revision` (P1): no conflict. Raddle splits the Sketch, as the Notes say.
- `claude-account-pick-simple` (T1): no conflict. It leaves "how a run goes on after a move" to this Motif, and D3 and D10 settle it. The restore target is already an Open risk.
- `machine-restart-shortcut` (F2): no conflict. The launcher reads the owner, which D5's registry keeps consistent.
- `hub-stop-button` (F5): no conflict. It is out of scope, and D7's Bolt `pause` hold does not contradict it.
- `per-machine-branch-prefix` (F4): none.
- Peer `testmap-payoff-measure`: none.
- Parent commits: `lym loom peers` lists none. Landed `dce431f4` (pause gate) does not conflict with D7.

**Decision notes**
- No `Decision for adopt-redesign:` note stands on the board list. None was unreachable.
- The record has no `Added after Discussion` entries.

**Tool calls**
- None. The board list, this task's entry, the peers and the touching entries were already in context.
