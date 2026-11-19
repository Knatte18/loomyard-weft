```yaml
lens: leanness
outcome: reported
```

Lens: leanness, a review of the decision record for content to move or cut.

**Findings**

1. **MEDIUM — Decisions, rationale lines that argue against alternatives (D8, D9, D11, D12).**
   - The spec says Decisions carry the decision and its rationale only, never a rejected alternative.
   - D8's first bullet explains why `Task.Revised` fails ("`stampDates` leaves `revised` alone on a status change, and it holds whole seconds"). That argues against a rejected option, and the support log's Rejected alternatives already holds it. The record only needs "covers every stored field, status included."
   - D9's rationale ends on "the launcher file keeps the token off process listings, which other users on the machine can read." The support log's "The token in the opened URL" entry already says this. The guarantee itself, that the opener's argument holds no token, already sits in the D9 bullet and its acceptance criterion.
   - D12's rationale line "a server an agent started would also carry the agent's limits into the operator's page" restates the rejected "set or clear the strand name" alternative.
   - Fix: trim each to the positive reason and leave the comparisons in the support log.

2. **LOW — D13, a decision that repeats D6 and adds nothing.**
   - Its last bullet, "`internal/fabricengine`, `internal/vscode`, `internal/ideengine` and `cmd/lym` change as D6 states", carries no requirement of its own. Cut it.

3. **LOW — D3, a trailing sentence about the server.**
   - "The board server never reads these files…" restates what D2 ("the board server serves the same embedded bytes") and D1 ("The board server (D9) serves the same projection, built by the same function") already settle. Keep it in one place, D2.

4. **LOW — Notes for the plan writer, the D7 note.**
   - The note explaining that D7 was dropped and its id left unused is worth keeping, since the carried Added-after-Discussion entries reference D7.
   - Its tail, "so the task changes no README layout", repeats the parent entry "The phone goal is met by today's README". Shorten it to the id explanation.

5. **LOW — Notes for the plan writer, the creel model note.**
   - It repeats D11's second and third bullets ("as creel's client starts creel's daemon", "as creel's daemon does").
   - Only `selfexec.New` is new information, and `relevant.md` already lists `internal/selfexec/selfexec.go` `New` and `Watcher.Replaced`. The note can go, or shrink to the `selfexec.New` hint.

6. **LOW — Auto-mode assumptions repeat the decisions.**
   - The D8, D9 to D11, D10, D11 and D12 lines each restate a decision body almost word for word. "D11: the server exits after 30 minutes… backs off to eight seconds" duplicates D11 outright.
   - The section is required, but each line could name only the self-picked choice in a few words, or point to the ledger's Q numbers, without restating the parameters.

7. **LOW — Open risks names its own resolution.**
   - "the record keeps them because the entry's Delivers line names them" already sits in the support log's Q8. The risk only needs the question to the parent.
   - Moving it is not required, since the risk must stand on its own for the reviewer. Emphasis only.

**Not findings, checked under the completeness-before-leanness test**

- The D3 and D6 bounds are long, but each carries mixed-version behaviour the plan writer needs.
- D9's header list and D10's route list are requirements.
- The carried Added-after-Discussion entries must stay verbatim.

Tool calls: none.
