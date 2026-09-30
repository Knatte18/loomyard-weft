# Support log — refusal-audit

## Interview

Autonomous session: no operator answered.
Each turn below is a question the discussion put to itself, the evidence read, and the pick.

1. **Is the task still needed after `webster-parent-merge-midrun` landed?**
   The dependency (`037c689eb`) fixed only the parent-merge ring.
   Reading the record-batch path showed more rings of the same shape: an OK report left on disk while record-batch refuses it leaves begin-batch (report exists) and recover-batch (OK report) refusing too.
   Pick: yes, the full audit is warranted.
2. **What does "way forward" mean precisely?**
   Without a definition the audit is not decidable per row.
   Pick: a named `lyx` verb, an automatic state transition, or a named `git` command on the warp; never hand-edited lyx state or a deploy (D1).
3. **Which refusals are in scope?**
   The board names four packages, but the gates it lists are raised in helper packages (`loomshed`, `loomengine`, `preflight`, `shedverbs`, `shedrun`).
   Pick: scope by the verb surface of webster, shed and loom, including those helpers; landing, batten, orch, fabric, burler and board are outside.
4. **Must every error return be a table row with a test?**
   Many returns are nil-dependency guards and raw I/O errors no run state can trigger.
   Pick: four classes (correctness, policy, transient, wiring); wiring and raw I/O are grouped rows with no per-row test (D2).
5. **Where does the table live?**
   There is no `manifest/designs/webster.md`; the documentation lifecycle puts a built module's design in its Go package header, and `webster-spec.md`'s charter excludes bracket-verb and audit internals.
   Pick: a new `contracts/specs/refusal-spec.md`, one section per module, linked from the webster, shed and loom docs (D3).
6. **Which fork-audit classes are policy?**
   Board: `fabric-reference`, `named-spawn`.
   Read the class comments: `nested-agent` cannot execute inside a fork; `parent-write` covers both a forged report and a harmless scratch write; `fork-contract-write` forges Master's judgment.
   Pick: D4's split, with `parent-write` split by path.
7. **What does "commits pass verify" mean for the downgrade?**
   Pick: re-run the batch's cards' own `verify:` commands in-process on the current worktree, only when a policy finding exists (D5).
8. **What happens to a correctness-class finding?**
   Refusing and leaving the report in place is the wedge.
   Pick: archive the report, record the batch failed, point at `recover-batch` (D6); the same disposition covers done-check and own-card bind failures.
9. **What about reports record-batch cannot attribute?**
   `ErrNoForkTranscripts` currently names a manual file move; `ErrNoBeginRecord` has the same ring shape.
   Pick: archive and re-drive with `begin-batch` (D7).
10. **Does a parent finding repeat?**
    The parent audit covers Master's whole session and nothing marks its findings consumed.
    Pick: disposition each finding once (D8).
11. **Drift findings about later cards?**
    Pick: warn on the current batch; the later card's begin-batch refuses with "edit plan, then rebaseline" (D9).
12. **Foreign plan edit mid-run?**
    `--fresh` rebuilds empty state, and run-entry validation over landed cards then refuses.
    Pick: a standalone `lyx webster rebaseline` verb (D10).
13. **Shed-level wedges?**
    `current_producer` naming a removed row (reachable after any deploy that renames a recipe row) and a re-blocking exhausted bounce budget have no verb out.
    Pick: generic `lyx shed goto` in `shedverbs` (D11), operator/orchestrator only.
14. **How is the principle kept after this task?**
    Pick: a CONSTRAINTS.md invariant enforced by review and per-row tests (D12).

## Rejected alternatives

- **Fix only the three named incidents.**
  Rejected: reading record-batch surfaced further rings of the same shape; the board asks for the whole audit.
- **Table inside `internal/websterengine/doc.go`.**
  Rejected: a Go comment does not render a table, and shed and loom rows do not belong in webster's package doc.
- **Table in `webster-spec.md`, or split across `webster-spec.md`, `shed-recipe-spec.md` and `loom-status-spec.md`.**
  Rejected: webster-spec's charter excludes internals, and loom-status-spec is a status-schema doc; one doc keeps the lookup single for the driver.
- **Stable refusal IDs embedded in every message, with a scan test matching code to table.**
  Rejected: touches every message for a tripwire the per-row tests already provide.
- **Keep correctness-class audit findings as hard refusals with a manual override verb.**
  Rejected: a failed batch plus the existing recovery ladder is a way forward with no new verb and no operator step.
- **Auto-downgrade every audit class to a warning.**
  Rejected: `fork-contract-write` and a Master write into the tree or reports dir can land unaudited or forged content.
- **Trust the report's OK status alone for the policy downgrade.**
  Rejected: the board requires the commits to pass verify, and the fork's account is what the audit distrusts.
- **`lyx webster run --rebaseline` flag instead of a verb.**
  Rejected: it would start Master outside loom; a standalone verb lets the loom Webster row re-step normally.
- **Recipe rename table so a deployed rename maps old producer names to new.**
  Rejected: covers only renames, not removals or exhausted budgets; `goto` covers all three.
- **Let ly-drive call `goto` itself.**
  Rejected: re-pointing a run is a human decision, the same reasoning the Transient Stop Invariant applies to re-steps.
- **Allow `goto` on a done run.**
  Rejected: re-opening a finished run risks a second landing; seeding a new run is the way forward.
- **Extend the merge-only head rule to accept fast-forwards onto parent commits.**
  Rejected: the dependency task decided that limit and its message already names a git recourse.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Status | Answer |
|---|---|---|---|
| 1 | Is the audit still needed after `webster-parent-merge-midrun`? | resolved (auto-pick) | Yes; further rings exist in record-batch. |
| 2 | What counts as a way forward? | resolved (auto-pick) | D1: lyx verb, automatic transition, or named warp git command. |
| 3 | Which packages are in scope? | resolved (auto-pick) | Verb surface of webster, shed, loom plus their gate packages; landing, batten, orch, fabric, burler, board out. |
| 4 | Must wiring guards and raw I/O errors get rows and tests? | resolved (auto-pick) | Grouped rows, no per-row test (D2). |
| 5 | Where does the table live? | resolved (auto-pick) | New `contracts/specs/refusal-spec.md`, linked from module docs; deviates from the board's "webster module doc" wording because the table spans three modules. |
| 6 | Is `nested-agent` policy class? | resolved (auto-pick) | Yes; the call cannot execute in a fork. |
| 7 | How is `parent-write` classified? | resolved (auto-pick) | Tracked tree or reports dir: correctness; elsewhere: policy. |
| 8 | What does "commits pass verify" mean? | resolved (auto-pick) | The batch's cards' `verify:` re-run in-process on the current worktree. |
| 9 | Where is a recorded warning kept? | resolved (auto-pick) | Batch state in `state.json`, the verb envelope's `warnings`, and a Go-appended `## Audit warnings` section in `summary.md`. |
| 10 | What happens on a correctness-class finding? | resolved (auto-pick) | Archive report, record batch failed, way forward `recover-batch` (D6). |
| 11 | What happens to an unattributable report? | resolved (auto-pick) | Archive, re-drive with `begin-batch` (D7). |
| 12 | How is a foreign plan edit accepted mid-run? | resolved (auto-pick) | New `lyx webster rebaseline` verb (D10). |
| 13 | How does a shed run leave a removed-row or exhausted-budget wedge? | resolved (auto-pick) | New generic `lyx shed goto` (D11). |
| 14 | May ly-drive call `goto`? | resolved (auto-pick) | No; operator and orchestrator only. |
| 15 | May `goto` re-open a done run? | resolved (auto-pick) | No. |
| 16 | How is the principle enforced after this task? | resolved (auto-pick) | CONSTRAINTS.md Refusal Way-Forward Invariant, review plus per-row tests. |
| 17 | Does loom's fresh-start coherence check misfire on a `goto` re-entry? | open | For the Plan's audit to decide per gate; recorded as an open risk. |
| 18 | Does `manifest/roadmap.md` carry this task? | open | Plan checks; the roadmap moves only if it is a planned item. |
