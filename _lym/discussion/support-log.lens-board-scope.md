```yaml
lens: board-scope
outcome: reported
```

Task: review the decision record and support log against the board (board-scope lens).

**Findings**

1. **MEDIUM**, class `decision`: Open risks / Scope "Out" (rewrite) and Auto-mode assumptions. Should the mode be built into the current `fabricengine`, or only written as a spec for the planned rewrite?
   - The entry's Relation section says the operator "wants this mode taken in if fabricengine is rewritten (note `clean-room-module-rewrite`)".
   - Note E5 `design-simplify-callers` wants `fabricengine` shrunk "first", because the operator finds it disproportionately large.
   - D1 through D9 add a second layout to nearly every fabric verb: clone, add, commit, push, pull, merge, teardown and a new view verb. That grows the package these two notes aim to shrink.
   - The record already lists this under Open risks. It is a question only the run's parent can settle, so it goes to the parent.

2. **LOW**, class `consistency`: Goal / D1, D3, D8, D9, D10. The entry says "only fabric knows where the weft lives". The record lets `lymcwd` ask the layout predicate (D1, D3). It also changes code outside fabric:
   - D8: PR-Review, Finalize and `lym beam pr` call the new view verb.
   - D9: verifytree, gatecli and other readers get the records-path helper.
   - D10: websterengine's write audit changes.

   No outside caller branches on the layout: D8 returns the branch itself in a two-repository hub, and D9 applies in both layouts. So the intent holds. But the Goal claims "only fabric knows which layout a hub has" while D1 gives `lymcwd` that knowledge too. State the `lymcwd` exception in the Goal or in D1 so the record does not contradict itself.

**Goal points of the entry**
- `_lym` and `.lym` are real directories, with no junctions or piggyback repo: met (D3, D4, D11).
- Same as today from outside, with only fabric knowing: met in substance. Wording caveat in finding 2.
- `.lym` is never committed and stays excluded, and `_lym` is not excluded: met (D3).
- Code branch or a separate branch for `_lym` commits: settled as the code branch (D4).
- What a landing does with `_lym` and `.lym`: settled. They are kept out of the parent by stripping at merge (D5), and pull requests use a records-free view ref (D8).
- Counterparts for the records branch, board, per-task branches and push transport: met (D4, D6, D8, D11).
- Whether code outside fabric (`hubforge`, `websterengine`, `burlerengine`) commits `_lym`: met. The support log's Q4 says none does. D10 and D12 cover websterengine and hubforge. burlerengine was verified to be layout-agnostic but is not named in the record. That is no finding.
- Relation to the rewrite: raised under Open risks, and finding 1 sends it to the parent.

**Entries, peers and parent commits checked**
- `clean-room-module-rewrite` (E2): possible scope conflict over timing (finding 1).
- `design-simplify-callers` (E5): tension with growing fabricengine now (folded into finding 1). No direct contradiction.
- `drop-compat-migration-code` (E6): no conflict. The record keeps today's two-repository behaviour and adds no migration code.
- `sandbox-closed-area-build` (F9): no conflict.
- Peer `plan-writer-explorer-forks`: none. It changes only a docs research path.
- Peer `testmap-uses-design`: none. It overlaps on the gate package only in text (D9 touches the green record).
- Parent commit `9c5bd13`, "Pause holds no push lock and skips a pair with no reed session": none. D6 keeps fabric's push lock and teardown lock as they are.
- Own entry: the record leaves the task's board entry untouched and files or promotes nothing.

**Decision notes**
- This task has no `Decision for fabric-single-repo-mode:` note on the board. None was unreachable.
- The record has no `Added after Discussion` entries.

**Tool calls**
- One call to `lym board get` for `design-simplify-callers` and `drop-compat-migration-code`. My context had their board-list briefs but not their bodies, which I needed to judge conflicts with fabric work.
