# Support log — twb-d0-ga

## Interview

### Turn 1 — exploration (autonomous; no operator)

Read the board entry and the related entries `plan-drift-refusal-route`, `weaver` and `discussion-premise-cites`; none plans the same change.
`plan-drift-refusal-route` concerns begin-batch's `delete-before-reference` refusal and its routing, a different check at a different verb.
Read `internal/websterengine`'s `doc.go`, `recordbatch.go`'s `postBatchChecks`, `cardgate.go`, `internal/planglyph/donecheck.go`, the head of `drift.go`, and the plan spec's Rename and gate-tier sections.
Read the `tender` plan's cards 10 and 12 from the weft history: card 12 spells the package rename as one file-rename pair per file, and card 10 Creates `plan:internal/pauseengine#…` handles.
Findings: the done-checks resolve every card's Create, Delete and Rename targets against the batch's final tree; `CardSHAs` holds only the report head; gate scopes are rendered at begin-batch from each card's target directories, so a moved-out directory stays in the batch gate.
Measured `lyx gate test ./cmd/lyx`: 8 s wall time on a warm cache.

### Turn 2 — questions and self-picks

- Q1. Resolve per card commit, or follow declared moves? Recommended: follow declared moves. Picked (A1).
- Q2. What declares a move? Recommended: file-rename pairs out of one directory that all land in one directory. Picked (A2).
- Q3. Which gates change for the move? Recommended: both the card gate (its own moves) and the batch gate (that card's and later cards' moves). Picked.
- Q4. Catch scan and name-pinned breaks by plan-time flagging or by running the package? Recommended: run it. Picked (A3, A4).
- Q5. Card gate or batch gate for the scan packages? Recommended: card gate, at 8 s per card. Picked (A3).
- Q6. Hard-code `./cmd/lyx` or configure? Recommended: a webster config key, set for this repo in its tracked layer. Picked (A3, A5).
- Q7. Should drift detection treat a declared move as expected? Recommended: no change; its exact-tier repair of a later reference is useful. Picked.

Weaver reuses begin-batch, record-batch and recover-batch, so it inherits D3–D5 with no dependency to name.

### Turn 3 — lens review by forks

Every lens returned a report.

- premises, MEDIUM: the rename card's own card gate already named `D'`, since `CardTargetDirs` adds only Rename new sides — fixed: D4 restated around the other targets in `D`, criterion replaced.
- premises, MEDIUM: no tracked worktree webster layer exists, and the repo default lives on the board — fixed: D5 and A5 name the board layer, pinned by the operator after landing.
- goal-scope, MEDIUM: the same layer ambiguity — merged with the premises finding, fixed.
- premises, LOW: `BindHandles` already matches renamed-to sides — fixed in D3 and the quarry risk.
- premises, LOW: no list-valued key exists — fixed: the key is a whitespace-separated string.
- goal-scope, MEDIUM: binding rests on an unchecked quarry premise — fixed: D3 matches created and renamed-to sides, and the risk names the remaining case.
- goal-scope, LOW: the scope guard is unaddressed — fixed: listed under Out, since a moved file is its Rename card's own target.
- goal-scope, LOW and leanness, LOW: the later-batch-move risk was muddled and repeated Out — fixed: cut.
- goal-scope, LOW: the docs-only extra step was unexplained — fixed: rationale names the docs scans in `cmd/lyx`.
- board-fit, MEDIUM: the key is Go-shaped where the .NET entries plan gate config — fixed: D5 says the key is Go-scoped and empty for other hubs.
- board-fit, LOW: the `plan-drift-refusal-route` exclusion gave no reason — fixed: Out names begin-batch's dispatch-time checks.
- board-fit, LOW: the Weaver relation — fixed: noted above.
- board-fit, LOW: the sibling bench-arm entries — rejected: bench arms of this task, not planned work.
- pattern-fit, MEDIUM: external-test units are not followed, and fixing that changes `PATTERN-glyph-conversion-chokepoint` — fixed: D2 names the change and adds the `_test` inverse.
- pattern-fit, MEDIUM: the refusal hedge and warn-versus-halt — fixed: a shape violation fails the load, like the model-spec check, since it guards what reaches a rendered command; a missing directory warns.
- pattern-fit, MEDIUM: the key's category was unstated — fixed: `CategoryGuard`.
- pattern-fit, MEDIUM and scenarios, MEDIUM: entries were unvalidated in a rendered command — fixed: D5's shape check and criterion.
- pattern-fit, LOW: `planparser` must stay quarry-free — fixed in Constraints.
- pattern-fit, LOW: `PATTERN-told-geometry` for D4's disk reads — fixed in D4.
- pattern-fit, LOW: how the key reaches the hub layers — fixed in D5 (the reconcile inherits the template value).
- scenarios, MEDIUM: a recovery re-render after the move drops `D` from an earlier card gate — fixed: D4's card-gate rule and criterion.
- scenarios, MEDIUM: a missing entry silently stops the scan — fixed: D5's bound states it, with a warning on every render and the plan-level verify as backstop.
- scenarios, MEDIUM: the root directory could be declared a move — fixed: D1 excludes root and nested moves, criterion added.
- scenarios, LOW: chained moves and a move into an existing `D'` — fixed: criteria added.
- scenarios, LOW: misleading findings after a partial batch — fixed: a finding names the moving card it followed.
- scenarios, LOW: config timing and slot load — fixed: D5 reads afresh per render; slot load is in Open risks.
- scenarios, LOW: `language: none` — fixed: D4 and a criterion.
- leanness, MEDIUM: the doc list appeared twice — fixed: Scope item 4 points at Constraints.
- leanness, LOW: the test-budget bullet restated CLAUDE.md — fixed: cut.
- leanness, LOW: the refusal-way-forward hedge — fixed with the pattern-fit finding.
- leanness, LOW: Auto-mode lines paraphrased Decisions — fixed: reduced to pointers.
- leanness, LOW: the 8 s figure was split across sections — fixed: moved into Open risks.
- leanness, LOW: Notes enumerated symbols — fixed: trimmed to the `tender` pointer.
- leanness, LOW: Turn 2 and the ledger repeat picks — rejected: the ledger records disposition, the turn records recommendations.

## Rejected alternatives

- Resolve each card's glyphs at its own commit: `CardSHAs` records only the head, quarry resolves a worktree, so each card would need its own checkout, and `PATTERN-verified-tree` forbids a detached checkout of the live worktree; it would also pass a card whose work a later card of the batch dropped.
- Batchifier puts a rename card in its own later batch: more batches and forks, and the rename batch's own gate still names the moved-out directory.
- Plan-Review or `validate-plan` flags a rename whose glyphs earlier cards use: the rename-last order is the correct plan, since earlier cards cannot name a package that does not exist yet.
- Plan-time flagging of name-pinned tests: the `gitrepoboundary` break was a method added by the card itself, and the tier-purity break a new `time.Sleep`; neither name exists in any test at plan time.
- `./cmd/lyx` in the batch gate only: catches the break one or more cards late, away from the card that caused it.
- Hard-coding `./cmd/lyx`: other hubs have no such package.
- A verb to accept a batch by hand: a guard bypass that the fix makes unnecessary for this case.
- Not following external-test units: leaves a false `create-not-done` for a Create in a moved directory's `_test` package, the same bug class.
- A worktree-layer value for `card_gate_packages`: it travels only with one task's records and reaches no other run.
- A list-valued key: no webster key is a list, and `--set` takes scalars.
- Warning instead of failing on a malformed entry: the entry is pasted into a command an agent runs, so a malformed one must not render.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 — resolved, auto-pick: follow declared moves within the batch (D3).
- Q2 — resolved, auto-pick: file-rename pairs out of one directory into one directory (D1).
- Q3 — resolved, auto-pick: both gates (D4).
- Q4 — resolved, auto-pick: run the packages, no plan-time flag (D5).
- Q5 — resolved, auto-pick: card gate (D5).
- Q6 — resolved, auto-pick: config key `card_gate_packages`, a string; this repo's value `./cmd/lyx` is pinned on the board layer after landing (D5).
- Q7 — resolved, auto-pick: drift detection unchanged (Scope, Out).
