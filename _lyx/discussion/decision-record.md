## Goal

Webster's Master reads only what every fork shares and what stays stable for the whole run: the plan's `00-overview.md`, the repo's rules when PATTERN is active (`PATTERN.md`, inlined by the pattern directive, and the `pattern/` background files it points at that touch what the forks will do) and what Claude Code already loads (`CLAUDE.md`).
It reads no source code, tests or other docs, because the forks rewrite them and every fork after the first would inherit stale reads.
Each implementer fork reads the code its own cards need, fresh, when the card starts; no stencil tells a fork it inherited code.

## Scope

In:

- `contracts/stencils/webster/webster-template-master.md`: the banner comment, the opening paragraph, the `## Orientation` section and `## What you never do` drop the read-the-codebase and inherit-my-reads framing, and name the Master's read set.
- `contracts/stencils/webster/webster-prefix-fork.md`: the title, the opening paragraph and the banner comment stop claiming the fork inherits a codebase orientation.
- `contracts/stencils/webster/webster-prefix-recovery.md`: its orientation steps match the fork's read set, and its "exactly like an in-session fork already holds it from Master" line is reworded.
- The live FRESH-READ fragments `webster-fragment-fresh-read-plain.md`, `webster-fragment-fresh-read-pack.md` and `webster-fragment-fresh-read-pack-card.md`: the "one Master read before forking you" framing is replaced.
- `contracts/stencils/webster/webster-body-verify-fix.md`: the line that says the fixer inherits Merriam's whole context states that this context holds no code.
- Docs that say the Master reads the codebase: `internal/websterengine/doc.go` (package opening sentence), `docs/overview.md` (the webster module line) and `tools/sandbox/SANDBOX-WEBSTER-SUITE.md`.
- Tests that pin the changed wording: `contracts/stencils/claims_test.go` and `internal/websterengine/template_test.go`.

Out:

- `webster-fragment-fresh-read-pack-inline.md` stays byte-identical, per its banner: it is the layout measurement baseline and changes only when the layout comparison is meant to change.
  Its "one Master read before forking you" sentence is a may-be-stale hedge, not a claim that Master read code, so it does not contradict D2.
- `contracts/stencils/pattern/pattern-directive-orchestrator.md` stays: its rules read is shared and stable, which is what this task keeps.
- No Go behavior change: no new hook, audit check or verb; the Master is held to D1 by its prompt alone.
- `tools/forkaudit` is not changed; the re-measurement runs after landing (D8).
- `webster-body-implementer.md`, the card-read fragments and the kick-start pack are not touched: the deviations rule stays with `webster-deviations-list`, and the test-file naming with `plan-test-file-path`, which depends on this task.
- Other roles' stencils: `stencil-intent-audit` covers the wider audit.

## Decisions

### D1 — Master's read set

**Decision:** The Master reads `{{.plan_dir}}/00-overview.md` once (task framing, Card Index, `## Shared Decisions`, `## Rename mechanic`, `## verify:`), and the `pattern/` background files the constraints section above it points at, when that section is present.
With card files banned, the Master applies that section's filter (what the forks will do) from `00-overview.md`'s Card Index and `## Shared Decisions` alone.
The template has no conditionals, so the wording names the rules through the constraints section and reads as vacuous when the pattern directive renders empty.
It reads no card file and no source, test or doc file outside the plan directory and those `pattern/` files, and runs no grep or quarry over the code.
The only other file it reads is the gate findings file a `Gate findings recorded at …` message names.
`## What you never do` gains a line forbidding any other read, with the reason in one clause: forks rewrite the code, so a read inherited by a later fork is stale.
Its existing line "You read files with Read and Grep" is narrowed to those files.
**Rationale:** Operator decision of 2026-10-15 on the board entry.
The plan and the rules are fixed during a run: `record-batch` and `recover-batch` refuse a drifted plan, and `begin-batch` refuses a plan that conflicts with the tree.
They are the only reads safe to hand to every fork.
`CLAUDE.md` needs no instruction: Claude Code loads it into the Master's context, and forks inherit it.

### D2 — Fork prefix framing

**Decision:** The fork prefix says the fork inherits the plan's framing and the repo's rules from Master, and no code: every file its cards need it reads itself, as the card steps below say.
The title drops "inheriting Master's context" for wording that does not imply a code orientation.
The `## You are the IMPLEMENTER` section, which tells the fork it inherits Master's loop instructions and must not drive them, is unchanged.
**Rationale:** The board entry asks for the inherit-my-reads framing to be dropped from the fork prefix.
The loop-instruction warning stays true and backs the `steerWebsterForkDeny` hook, which is the deterministic guard.

### D3 — FRESH-READ fragments

**Decision:** In each of the three live FRESH-READ fragments, three spans are replaced: the sentence "Inherited context can be stale.", the sentence "A file you looked at during orientation — your own, or one Master read before forking you — is not necessarily the version on disk right now: …", and the sentence pair "Only your own reads, taken now, are current; content you merely inherited is not."
One replacement statement covers all three: no code reaches you from Master, the plan and rules you hold stay current, except a rules file a card of this run targets, which you read fresh like code, and only your own reads of code taken in this batch are current.
In `webster-fragment-fresh-read-plain.md` the verb "re-read" becomes "read", so the text holds for the cold recovery strand, which renders that fragment too and inherits nothing.
Every other sentence of each fragment is unchanged, apart from the plain fragment's re-read to read.
**Rationale:** "One Master read before forking you" presumes Master read code; after D1 it never does.
The per-card layout of `kickstart-per-card` then becomes the fork's whole code read set, as the board entry intends.

### D4 — Recovery prefix

**Decision:** The recovery prefix's orientation reads `_lyx/plan/00-overview.md` and the `pattern/` background files the recovery prompt's own constraints section points at: the implementer directive's set, filtered by what the strand is about to change.
The code its cards need it reads per card, as the shared implementer body says, not as an up-front orientation step.
The closing line "they assume you already hold this orientation, exactly like an in-session fork already holds it from Master" is reworded to say the recovery strand now holds what a fork inherits.
**Rationale:** That closing line claims an in-session fork holds a codebase orientation from Master, which D1 makes false.

### D5 — Verify-gate fixer

**Decision:** `webster-body-verify-fix.md` keeps "You inherit Merriam's whole context, so this prompt is deliberately thin" and adds that this context holds no code, so the fixer reads the code behind each finding itself.
**Rationale:** The fixer is a fork of the same Master and edits code after every batch has committed; it must not assume an inherited view of that code.

### D6 — Docs

**Decision:** The three docs in Scope say the Master reads the plan and the repo's rules once, never the code, and each fork reads its cards' code fresh.
**Rationale:** CLAUDE.md's docs-in-the-same-commit rule and `PATTERN-documentation-lifecycle`; each currently states the up-front codebase read as the design.

### D7 — No new PATTERN entry

**Decision:** The rule lives in the Master template and `websterengine`'s `doc.go`, not in `PATTERN.md`.
**Rationale:** It binds one role, webster's Master; it is not cross-cutting.

### D8 — Re-measurement hand-off

**Decision:** No card implements D8.
The carrier is the change description: this Decision is a check to run by hand after landing, and the Describe row, which runs in every loom run and reads this record whole, lists it under what to check or run by hand, so it reaches the pull request body and the landing commit's message on main.
The check: `tools/forkaudit` over the first Webster runs made on a binary deployed from this task's landing commit, comparing per-fork total reads and failed Edit calls (the stale-read proxy) against the 2026-10-12, -13 and -15 audits, and not calls before the first edit, plus the Master's own reads in those runs as the compliance check for D1.
Forks explore more by construction, so a rise in per-fork reads is expected and is not a regression.
A board note for the check has kind `note` and label `audit`, as `suite-weight-quiet-rerun` has, names this task's slug, and is due at those first runs.
What the carrier can lose: the description reaches an actor only at PR-Gate.
When the orch reviews the pull request, its board policy puts the check on the board as such a note;
a run the alignment agent approves lands with no orch turn, so the check then survives only in the landing commit's message, read by no scheduled step.
The carrier depends on no parent, so a run with no recorded parent carries it the same way.
**Rationale:** The board entry asks for the re-measure "afterwards"; it needs runs on the deployed binary, so no card can run it.
The task summary cannot carry it: the Master writes `summary.md` strictly from batch digests, so no plan or card text reaches it.
No session of this run that reads the record may write the board, and none is told to message `ly:orch` about it, so neither can file the note.
The Describe row is the one scheduled reader of this record whose output outlives the run.

## Constraints

- `PATTERN-stencil-ownership`: stencils stay files read through `stencilstore` at call time; no wording moves into Go, and no new fragment is added.
- `PATTERN-parent-directive`, `PATTERN-edit-directive`: the master and recovery templates keep their markers, and no stencil gains an ask-the-operator line.
- `PATTERN-producer-pointer-rule`: no stencil paraphrases the plan spec or another producer's contract.
- `PATTERN-test-economy`: updated pins extend the existing `claims_test.go` and `template_test.go` rows.
  They assert only what must hold (the Master's read set is named, other reads are forbidden, no stencil claims the fork inherits code), not full sentences, in line with the operator's 2026-10-15 decision on `lighter-test-rules`.
  A new top-level test is added only if no row covers the master's read set, and then a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- `PATTERN-comment-line-breaks` and the semantic-line-break rule: the `doc.go` and every `.md` edit keep one sentence per line.
- The master template keeps no `{{if}}`/`{{range}}` and every marker it has; the fork prefix keeps `{{.outcome_path}}`, `{{.summary_path}}` and `{{.friction_directive}}`.
- Each stencil keeps its leading banner comment, updated where it states the inheritance claim.

## Auto-mode assumptions

- The inline-layout fragment stays untouched (its banner's baseline rule wins over uniform wording).

## Open risks

- A plan whose cards edit `PATTERN.md`, a `pattern/` file or `CLAUDE.md` leaves the Master's copy stale for later forks.
  The bound: a card that edits such a file names it as a target, so the fork that edits it reads it fresh, and a later fork that only obeys the changed rule gets it through the plan's `## Shared Decisions`.
- The Master may still read code despite the prompt, and would hand those reads to later forks as stale context.
  The bound: under D3's fresh-read rule each fork reads every file it edits itself before the first edit, so a stray Master read costs context, not correctness.
  The post-landing check of D8 detects it.
- A Webster run in flight at deploy mixes the two designs, since stencils are read at call time: a Master that read code under the old template forks under the new fork prompt, or a resumed Master gets the new template.
  The fresh-read rule makes forks read their own files either way, so the mix costs accuracy of wording, not correctness.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d8-carrier-task-summary-false-premise (class decision, severity MEDIUM, open in rounds 1, 2, 3)
  - d8-writer-orch-edge-unbounded (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- The rendered master prompt tells the Master to read `00-overview.md` and the rules, and contains no instruction to read the codebase, its structure or its conventions, and no claim that forks inherit Master's code reads.
- The rendered master prompt forbids reading any file outside the plan directory, the `pattern/` background files and a named gate findings file, and names the stale-inherited-read reason.
- The master prompt rendered with an empty `pattern_directive` (no `PATTERN.md`) contains no constraints section, and every mention of the `pattern/` background files in it is conditioned on that section being present; it names no read beyond the plan directory and a named gate findings file.
- The rendered fork prompt in the per-card, file and pack-off layouts says no code reaches the fork from Master, and no fork prompt says the fork inherits a codebase orientation.
- The rendered recovery prompt orients from `00-overview.md` and the rules, reads code per card, and its fresh-read rule holds for a session that inherited nothing.
- The rendered verify-fix prompt says the inherited context holds no code.
- The inline-layout fragment is byte-identical to its current text.
- The three docs in Scope no longer say the Master reads the codebase.
- `lyx gate test ./contracts/stencils/... ./internal/websterengine/...` passes with the updated pins.

## Notes for the plan writer

- Wording pins on the changed text sit in `contracts/stencils/claims_test.go` (the fresh-read and fork-prefix claim rows) and `internal/websterengine/template_test.go` (the fresh-read bodies).
- `template_test.go` also pins the master's `## Orientation` heading in an order check against the PATTERN directive; keep the heading or update that check.
