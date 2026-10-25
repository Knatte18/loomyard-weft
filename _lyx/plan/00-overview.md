---
format: 5
approved: true
language: go
---

# Plan: Discussion scope check in lenses, parent-only questions to the parent, Parent-Align

Move loom's Discussion scope check from the alignment agent into the Discussion-Review perch's lenses, send a `decision`-class finding to the run's parent through the Discussion-Bouncer instead of the fixer, give the darn writer the same outdated-entry check, and rename loom's `Merge-In` row to `Parent-Align`.
Cards 1 to 6 remove the parent-review gate, its verbs, the alignment agent and their config keys, keeping archived records readable by reports; cards 7 to 14 add the scope lenses, the class definition, the writer rules and the Bouncer's `decision` escalation; cards 15 to 18 rename the row.

## Card Index

1 — decision-note-title — move DecisionNoteTitle into orchengine so it outlives internal/alignment
2 — report-archived-records — give reportengine its own read-only decoders of archived parent-review and alignment records
3 — drop-parent-review-gate — remove the parent-review gate kind, Env.ParentReview and the loom wiring of the gate and the aligner
4 — drop-review-verbs — remove lyx loom review, the status waiting note and batten's ReviewWait seam
5 — delete-alignment-and-parent-review — delete the two packages, the loomengine prompts and the five stencils
6 — retire-alignment-keys — drop the four loom.yaml keys from Config and the template, and retire them through MigrateConfig
7 — scope-lenses — rewrite goal-scope and board-fit as procedures and add them to review-discussion
8 — discussion-rubric-decision — define decision in the Discussion-Review rubric as a question only the parent can settle
9 — discussion-writer-stencil — drop the parent-review text and leave parent-only questions open in Open risks
10 — darn-writer-entry-check — the darn writer checks its entry and asks its parent when it is plainly outdated
11 — carry-over-added-since — a carried finding names the Added after Discussion entries recorded since its escalation
12 — bouncer-decision-escalation — the Bouncer escalates a decision-class finding to the parent with cause decision
13 — discussion-decision-switches — the escalate_decisions and decisions_escalated row keys, set on loom's Discussion rows only
14 — decision-cause-readers — the arbiter refuses a decision escalation and the circling verbs name the cause
15 — landing-fix-attempts-key — rename merge_fix_attempts to parent_align_fix_attempts with its migration
16 — parent-align-row — rename the Merge-In row, producer, engine key, fixer mode and gate key, and carry runs across
17 — parent-align-prose — rename Merge-In in docs, specs, pattern background files and comments
18 — orch-stencils — drop parent review from the orch prompts and name decision escalations and Parent-Align

## Rename mechanic

1. For a file pair (both sides file self glyphs or paths), run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.
5. For a symbol pair, run the hub's structure-aware rename tool the implementer prompt names (`language.yaml` `rename_tools`), never a text or regex replace, then grep for zero remaining old-name references and confirm the symbol's doc comment opening names the new name.

## Shared Decisions

### Decision: decision-note-title-home

- **Decision:** `DecisionNoteTitle` moves to `internal/orchengine`, which renders it into the orch role, and `internal/loomcli` imports it there.
- **Rationale:** both remaining callers may import `orchengine`, and `boardengine` would drag `fabricengine` into `orchengine`'s import set.
- **Applies to:** card 1

### Decision: hand-renames

- **Decision:** the hub's `language.yaml` lists no `rename_tools`, so symbol renames are `Create` handles plus edits of every caller, and file moves are `Rename` file pairs.
- **Rationale:** the plan gate refuses a symbol `Rename` pair without a rename tool.
- **Applies to:** cards 15 and 16
