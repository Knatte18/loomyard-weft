## Interview

Autonomous mode: no operator answered; every pick below is the agent's own, recorded in the Question ledger.

1. Read the `board-model` entry: the operator decided the two-kind model, labels with type folded in, config-held lists, README grouping, body-from-file and a mechanical intake verb on 2026-10-04.
   Two questions were left open: migration of existing entries, and the intake's repo and auth.
2. Read the rest of the board.
   `cli-tree-exceptions` lists two `board notes` findings that D5's alias removal resolves; `card-batching` is the one note with a `depends_on`, which D12's rule moves into its body as the entry asked.
   `config-overlay` confirms config is per worktree today, which shapes the risk in D3.
3. Explored `boardengine` (store, task, render, legacy migration, config), `boardcli` (verbs, aliases, text listing), `selfreportengine` (hardcoded `targetRepo`, `NewGitHubClient` seam), `githubclient`, and the other `boardengine` importers (batten, ide, landing).
   No other module reads `Tier` or `Type`; batten reads only `Recipe` through a path-only `Board`.
4. Settled the migration (D12) from the entry's own proposal, adding the brief-prefix parse, the type-name prefix drop and the note-dependency move.
5. Settled the inbox repo and auth (D11): reuse `selfreport`'s constant and client seam.
6. Designed the intake verbs (D10) around the hand intake's three moves, with board-first sequencing and an idempotent retry for a failed close.
7. Checked every decision against PATTERN: config-strictness, github-auth, cli-cobra, test-tier-purity, documentation-lifecycle.

## Rejected alternatives

- Label vocabulary in a hub-level file under `_board/`: consistent across worktrees, but bypasses `configengine` and contradicts the operator's "in config" decision; per-worktree drift is recorded as a risk instead.
- Separate `type` and `labels` fields: the operator decided one classification; a single field with two config lists keeps the axes without two fields.
- A configured type-to-heading mapping: yaml maps lose order, and a derived heading covers the decided names.
- An explicit `lyx board migrate` verb: every hub would need an operator step; on-load conversion has precedent in the legacy migration.
- Stripping nothing from briefs on migration: briefs would repeat the labels.
- Merging near-duplicate area labels during migration (`perf`/`speed`, `models`/`engines`): curation is the operator's call.
- Intake GitHub code in `boardengine` or a new `intakeengine` package: the inbox repo constant already lives in `selfreportengine`, and a new module adds docs and sandbox obligations for three calls.
- Intake under `lyx selfreport`: the verb's output is board entries, and the operator named it a board intake.
- Closing the issue before the board write: a failed board write would lose the issue from the inbox.
- Keeping the `notes`/`promote-note` aliases: they reach every entry under a name that now means one kind.
- Validating only changed labels on a patch: lets an orphaned label survive forever; full-entry validation forces cleanup and names the way forward.
- A `--kind` filter: not asked for; the text listing shows kind.

## Review rounds

### Parent review, round 1

- F1 (BLOCKING, D13's label list never reaches the live config): fixed.
  Confirmed `_lyx` in the pair links to the pair's weft worktree.
  D13 now has the plan put the exact `labels` list in the PR description for the hub to write into the parent's live `board.yaml` before deploying; an Open-risks item and an acceptance criterion record it; no weft-carry mechanism is added.

## Question ledger

- Q1 (resolved, auto-pick): how existing entries migrate. Pick: on-load conversion per D12. Why: no operator step per hub, mirrors the legacy migration, loses nothing.
- Q2 (resolved, auto-pick): which repo intake reads and how it authenticates. Pick: `selfreport`'s target through `githubclient.New` (D11). Why: the inbox is where selfreport files.
- Q3 (resolved, auto-pick): where the label lists live given config is per worktree. Pick: `board.yaml` via `configengine`, as the operator decided; drift is an open risk.
- Q4 (resolved, auto-pick): does a task need a type label. Pick: one or more. Why: it carries its notes' types, and a typeless task would vanish from type-based views.
- Q5 (resolved, auto-pick): where `undecided` lives. Pick: the `labels` list.
- Q6 (resolved, auto-pick): validation scope on a patch. Pick: the full resulting entry.
- Q7 (resolved, auto-pick): how `--label` filters combine. Pick: AND.
- Q8 (resolved, auto-pick): remove the `notes` and `promote-note` aliases. Pick: remove (D5).
- Q9 (resolved, auto-pick): how a failed GitHub close recovers. Pick: re-running `import` on a recorded open issue only closes it.
- Q10 (resolved, auto-pick): what happens to a note's `depends_on` on migration. Pick: a closing body sentence, list emptied.
- Q11 (resolved, auto-pick): how loomyard's migrated area labels pass validation. Pick: the PR description carries the list, and the hub writes it into the parent's live `board.yaml` before deploying (D13, revised after parent review round 1).
