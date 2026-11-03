```yaml
lens: board-scope
outcome: reported
```

Lens board-scope: checking the decision record against the board entry, the touching notes and the peer run.

**Findings**

1. MEDIUM — Open risks / D9. The record says nothing about the peer run `decision-record-on-board`, which edits the same Discussion-Bouncer row that `cartoon` copies.
   - The peer's record (Discussion-Write, now written) adds `approve_seam: discussion` to tapestry's Discussion-Bouncer in `contracts/recipes/tapestry-recipe.yaml`.
   - Its D7 adds a guard test: every Bouncer row of every embedded loom recipe that names the decision record must carry that key.
   - Its Open risks names this task outright and asks the parent to confirm the coordination: "whichever lands second carries the key into `cartoon`".
   - Our D9 equality pin, `cartoon` rows equal tapestry's Discussion rows, does keep the two copies in step.
   - The problem: `cartoon` declares no `contract: loom`, so the peer's guard over "embedded loom recipes" may not cover it. A `cartoon` added after the peer lands would then be caught only by our pin, and our pin only if it compares config keys, not just names and edges.
   - Fix: add an Open risks line naming the peer and the shared row, and say in D9 that the pin compares each row's whole config, so `approve_seam` is included.
   - Class: consistency. The writer can settle it; it is not a parent decision.

2. LOW — D10 acceptance, "tapestry's recipe file is unchanged". The peer task changes that file, so read literally the check can fail after a rebase onto the peer.
   - Evidence: peer record line 77, "The Discussion-Bouncer row in `contracts/recipes/tapestry-recipe.yaml` gains `approve_seam: discussion`."
   - Fix: reword to "this task makes no change to tapestry's recipe file".

**Goal points of the entry**

- The block names a recipe; Done when the sub-run ends done, Stuck when it stops: met (D1, D4).
- Inline: same process and status file, history entries marked with their block, no second run or driver: met (D2, D6).
- Pause, goto and review budgets work as in any run: met (D3, D7).
- Entry gate: a check with no agent, entering or passing over the block whole, recorded as passed over: met (D3).
- The gate is named in the recipe from a set lym ships, with at least one that always enters: met (D5, `always`).
- `cartoon` as a recipe of its own holding tapestry's Discussion rows: met (D9, block source only).
- A new recipe beside tapestry that runs `cartoon` through the block and then tapestry's Plan and build rows; tapestry unchanged: met (D10).
- Discussion proposes the new recipe's name and the operator approves it: met. `arras` is proposed and flagged in Open risks for the parent.
- Out items (the Bolt-reading gate check, the bench): kept out (Scope).
- Open questions on goto, status, progress, segment budgets, and keys and profiles: settled (D3, D6, D7, D8).

**Entries, peers and commits checked**

- Note `discussed-task-split`: no conflict. Running `cartoon` alone and storing the record stay in that note, and the record keeps them out.
- Task and peer `decision-record-on-board`: overlaps on the Discussion-Bouncer row (finding 1); no design conflict otherwise. Its store is decided by the recipe's rows, so it also works on an `arras` run.
- Note `parts-plan-and-build`: no conflict.
- Parent commits: `lym loom peers` lists none.

**Decision notes**

- No `Decision for shed-sub-run-block:` note exists on the board.
- No `Added after Discussion` entries exist in the record.

**Tool calls**

- `lym loom peers`, to refresh the peer list: my context showed the peer with no decision record yet, and it now has one.
- A grep of the peer's decision record (`decision-record-on-board/_lym/discussion/decision-record.md`), to check for overlap with `cartoon` and the Discussion-Bouncer row: my context did not hold that record.
