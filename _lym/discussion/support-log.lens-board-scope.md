```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked whether the decision record fits the board entry, the project, the other entries and the peers.

**Findings**

1. **MEDIUM, class `decision`, about D9, D10 and D11 and the Open risks line on trial order.** The record builds the mutation check before the trial has run. That reverses the board entry's order and drops its condition.
   - The entry says: "1. Trial first: replay the step by hand over about twenty landed diffs… 2. If the trial pays: a gate report step…". It also says the check "starts with its hand trial", and the project body says "starting with its hand trial".
   - The record builds the planner, the step and `lym gate mutate` in this run. It moves the trial out to a board note (D11).
   - That spends the study's estimated 3 to 4 darns before anyone knows whether the check pays, which is the cost the entry's order was meant to avoid.
   - The record raises this under Open risks, but only the parent can settle it. Question for the parent: should this run build D9 to D11 now and ship the step unlisted? Or should the trial run first, as its own darn, with D9 to D11 cut from this run until its counts are in?

2. **LOW, class `scope`, about D11.** If the trial note is never taken up, the shipped `mutation` step and `lym gate mutate` stay in the code with no gate running them.
   - Board note E7 `drop-unused-features` (under project `code-cleanup`) exists to find and remove features like that.
   - This is not a contradiction today, but it is a tension the parent may want to weigh in the question above.

3. **LOW, class `scope`, about D8.** The entry asks for a lookup "over the landings since the last green". D8 does not find the last green itself: the caller must pass it with `--since`.
   - The support log (Turn 2) says lym keeps no "last green" record for a parent branch. That is a reasonable basis, and the entry does not demand an automatic lookup.
   - Still, the parent or a fixer now has to know the last green commit to use the verb. The record could say who supplies it in practice, for example from the Flaky test failures section of a landing commit or from the parent's own record.

**Goal points of the entry**

| Goal point | Met? | Where |
|---|---|---|
| Reach note: a reader of the maps with an index | Met | D1 |
| Reach note: a changed-function finder that compares function texts | Met | D2 |
| Reach note: a note in code review, ranked and capped, with unreached functions and the tier 2 or spawn caveat | Met | D4, D5 |
| Reach note: the writer gets the note from its own verify gate once the tiers pass | Met | D3, D12, with `batch` for tapestry writers |
| Reach note: a read verb a writer can call before editing | Met | D6 |
| Reach note: no upkeep per landing; stored map plus a fresh map of the diff's packages | Met | D3, D1 (the index is written only by the on-command `--map` run) |
| Triage: a line in the gate's failure report and in the fixer's brief | Met | D7 |
| Triage: for a red parent, the landings since the last green whose changes the test reaches | Met, with the caller supplying the last green | D8, see finding 3 |
| Triage: reads through the shared reader | Met | D7, D8 |
| Mutation: hand trial first | Deferred to a board note | D11, see finding 1 |
| Mutation: a gate report step only if the trial pays | Built anyway, but listed in no key | D10, D11, see finding 1 |
| Mutation: mutants only in new or changed blocks, capped per diff, uncovered blocks reported without mutants, survivors to review with file, line and operator | Met | D9, D10 |
| Mutation: a survivor only another package's test could kill is discounted through the reach note | Met | D9 |
| Build order: reach note, then triage, then mutation | Met | Goal section |
| Leave this task's own board entry alone | Met | D11 files a note, not a task, and touches no entry body |

**Entries, peers and parent commits checked**

| Item | Conflict? |
|---|---|
| P3 `testmap-uses`, the project | None. It is consistent with it, apart from the trial order in finding 1. |
| T1 `testmap-use-cases`, done research | None. It is the source. |
| P1 `code-cleanup` and E7 `drop-unused-features` | None as a contradiction; see finding 2. |
| Peer `pause-after-crash-fixes` (fabricengine, tender, pauserecord) | None. No overlap with gatecli, testcover or loomcli review notes. |
| Parent commits | None. `lym loom peers` lists none. |

**Decision notes**

No `Decision for testmap-uses-design:` note is on the board, and the record has no `Added after Discussion` entries. There was nothing I could not reach.

**Tool calls**

None. My context already held `lym board list`, `lym board get` for this task, the project and the research task, and `lym loom peers`.
