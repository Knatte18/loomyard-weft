# Plan cards: headroom check, Extends label, an amendment owner, failures traced to cards

`plan-card-rules` · Task · enhancement, loom, webster, burler, orch, quarry · running · Webster-Bouncer

## Decision (operator, 2026-10-22)

Four notes merged into one loom task about the plan and its cards.
Scope:

1. The plan gate compares the plan's declared `PATTERN.md` growth against the headroom line and refuses a plan that exceeds it.
2. The `Extends` card label, its card-DAG rule and its plan-review check; the `tools/plandag` measurement comes first, as below.
3. A parent decision that amends a plan card has an owner inside the run.
4. Enclose's first use only: a verify failure at Merge-In or Publish is mapped to its member glyph and the plan card that targets it, and the fixer's brief names that card.

## Headroom check (from note plan-gate-pattern-headroom)

Imported from [issue #735](https://github.com/Knatte18/loomyard/issues/735).

A plan-stage burler round (plan, round 2) had to measure by hand whether the plan's declared `PATTERN.md` growth fit the headroom line.
The round's finding asked for a named cut of at least 370 bytes across `PATTERN.md` entries outside the three the plan declares, keeping every rule, marker and link.
Measuring each candidate gave wording-only cuts of 152 bytes, and the overview's entries are already terse, so the proposed fix could not be carried out.

This is abnormal because the plan declares more `PATTERN.md` growth than the headroom line admits, and neither Discussion nor the plan gate catches that before a burler round.
The check is a mechanical byte comparison, yet it fell to a reviewing agent to measure it manually, costing a round and leaving an unsatisfiable finding.

Suggested fix: make the plan gate (or the Discussion stage) compare the plan's declared `PATTERN.md` growth against the current headroom and refuse or flag a plan that exceeds it, so the plan is corrected before review.

Provenance: task `batten-start-speed`, from note `burler-plan-r2.md`.

## Extends label (from note plan-extends-label)

### What was seen

`tools/plandag` (landed `c03af73d0`) was run on nine recorded plans of 12 to 20 cards (2026-10-18, plan artifacts from the records repo).
The best parallel speedup, cards over critical path with every card counted equal, came out at about 1.8x median:

| Plan | Cards | Critical path | Best speedup |
|---|---|---|---|
| `overview-seats` | 20 | 11 | 1.8x |
| `webster-fork-stencil-fixes` | 17 | 9 | 1.9x |
| `batten-start-speed` | 16 | 9 | 1.8x |
| `claimed-scope-edit` | 14 | 8 | 1.75x |
| `run-pane-timing` | 14 | 5 | 2.8x |
| `engine-registry` | 14 | 11 | 1.3x |
| `engine-model-choice` | 14 | 8 | 1.75x |
| `shed-tackler` | 14 | 7 | 2.0x |
| `report-timing` | 12 | 9 | 1.3x |

Much of each chain comes from a few shared spots that many cards change by adding to them, not by changing what is there:

- docs: the package `doc.go` (4 to 7 cards per plan), `docs/overview.md` (3 to 5), `PATTERN.md`;
- specs: `contracts/specs/refusal-spec.md` (4 to 5 cards);
- CLI wiring: `loomCLI.wire`, `battenCLI.wire`, `fabriccli#addWeftVerbs` (3 to 6 cards), where each card adds a flag or a verb;
- `cmd/lyx/testdata/test-budget.yaml` (7 to 9 cards in three plans), since removed by `test-budget-remove`.

Real overlaps also exist, such as six cards changing `websterengine#Run` in `engine-model-choice`; those stay dependencies.

### Idea (agreed with the operator 2026-10-18)

A plan card states how it changes a shared spot, not only that it changes it:

1. A new card label, working name `Extends`, beside the change labels and `Uses`: the card only adds to the ref (a flag, a list line, a doc paragraph, a spec entry) and changes nothing already there.
2. Two `Extends` on the same ref are no dependency in the card DAG; `Extends` against a real change still is.
3. Git merges additions in different hunks by itself; two additions on the same lines are a simple conflict for the merge-back resolver of parallel batches (see note `parallel-batch-parts`).
4. Plan review checks the label: an `Extends` on a ref the card in fact rewrites is a finding, like a missing `Uses` today.
5. Docs use `Extends` too, so the card that writes the code also writes its docs, and no separate final docs card is needed.

The risk is a wrongly labelled `Extends`, which surfaces as a harder merge conflict at merge-back, never silently.

### First step: measure before building

Before the plan format changes, extend `tools/plandag` (darn, review 0) to recompute the DAG with additive refs guessed by path and name (doc files, `docs/overview.md`, `PATTERN.md`, `contracts/specs/*`, CLI wire functions), treating change-against-change on them as no dependency.
Rerun it on the plans above and compare critical paths; the size of the drop decides whether the label is worth building.

#### Rough estimate (orch, 2026-10-18)

A throwaway awk pass over the plandag matrices recomputed each critical path with plandag's own overlap rule; with nothing skipped it reproduced every critical path above.
Columns skipped for change-against-change only: `test-budget.yaml` (already gone), then doc files, `docs/`, `PATTERN.md` and `contracts/specs/`, then also wire functions guessed by name (`wire*`, `addWeftVerbs`).
Use against change still counts.

| Plan | Cards | Today (no test-budget) | Docs and specs additive | Plus wire additive |
|---|---|---|---|---|
| `batten-start-speed` | 16 | 9 | 8 | 8 |
| `claimed-scope-edit` | 14 | 8 | 8 | 6 |
| `engine-model-choice` | 14 | 8 | 7 | 6 |
| `engine-registry` | 14 | 7 | 5 | 5 |
| `overview-seats` | 20 | 11 | 9 | 8 |
| `report-timing` | 12 | 4 | 3 | 3 |
| `run-pane-timing` | 14 | 5 | 5 | 5 |
| `shed-tackler` | 14 | 5 | 5 | 4 |
| `webster-fork-stencil-fixes` | 17 | 8 | 8 | 8 |

Median best speedup, cards over critical path with every card counted equal: about 2.0x today, about 2.5x with docs, specs and wire functions additive.
Removing `test-budget.yaml` already lifted the median from about 1.8x.
Docs and specs alone give the smaller part; wire functions give about as much again.

### Side finding

`fabriccli#addWeftVerbs` holds shared hidden flags and pre-run logic beside the verb list, which is why so many cards touch it; a thin wire function, only the list, with flags and logic beside each command, would remove much of that hot spot.
Orch view, not a decision: this is a separate code cleanup, to become its own note if the measurement shows wire functions matter.

## Card amendment owner (from note plan-card-amendment-owner)

Imported from [issue #806](https://github.com/Knatte18/loomyard/issues/806).

### What was seen

In the quarry hub, run `csharp-alphabet` blocked at Webster on a plan defect (see #805).
The arbiter handed on, and the orch recorded a parent decision to amend card 9 and rebaseline.
No role could carry the decision out: the driver edits no plan files, Webster's master edits only its contract files, the recovery strand will not edit the approved plan, the orch role forbids editing `_lyx/plan` in a task pair, and `lyx webster rebaseline` says the operator names the cards.
The run sat parked until the operator was asked, and the operator objected that the agents should have fixed it themselves.

### Workaround

On the operator's order the orch edited card 9 and ran `lyx webster rebaseline --card 09` and `lyx loom resume`.
A hub orch rule now lets the orch make such an edit without asking.

### Expected

A parent decision that amends a plan card has an owner inside the run, such as the arbiter, a plan-fix row or the orch by role, so the run goes on without the operator.

## Failures traced to cards (from note enclose-checks)

Operator, 2026-10-08: quarry gains `Enclose`, written as quarry's task `enclose-locations` (in the quarry repo; the operator spawns it).
It maps `file:line` ranges from tool output to the innermost enclosing member glyphs, with the same Symbol data `Resolve` returns, also at a git revision (`quarry enclose [--rev R]`).
Markdown is out of its scope: a `.md` location is a per-item rejection.

### Change of direction, 2026-10-17

The original use was a kick-start pack of the failing members for recovery agents and fixers.
The pack measurement this waited on (operator decisions 2026-10-11 and 2026-10-15) came in with `docs/research/glyph-audit-2026-10-17/`: forks read code themselves whatever the pack holds, and note `kickstart-pack-remove` records the operator decision to remove the pack.
A pack of failing members is dropped with it: a fork given `file:line` reads around it in one call.

Operator view, not a decision: Enclose can be useful for mechanical checks, where Loomyard's own code needs to know which member a line belongs to, with no agent in the loop.

### Orch recommendation, not a decision

First use, which changes no flow: a verify failure at Merge-In or Publish is mapped through Enclose to its member glyph, then to the plan card whose targets hold that glyph and the batch and commit that ran the card (`_lyx/webster/state.json`).
The fixer's brief names that card, its intent and its commit, as a starting point instead of the whole diff.
Later possibilities, each its own decision:

- Re-run only the batch that made the failing change (`lyx webster reset --batch` and a fresh fork with the failure attached), instead of a whole-diff review round.
- A finding that cites `path:line` is mapped to a glyph at the reviewed revision, so later edits never shift it, and it can be checked against the plan's targets.
- A failure in a member no card targets is reported as a scope gap: the plan missed it.
- Test, vet and lint failures are grouped per member, so duplicate findings fold.

Measure that a fixer uses the named card before building more, as the pack showed agents may read and ignore such hints.

Glyph↔path conversion stays in quarry; Loomyard never intersects lines with symbol spans itself.

### Quarry's Enclose as delivered (quarry PR #36, 2026-10-08)

- Released in quarry v0.3.0: `(*Repo).Enclose(locs)` and `(*Repo).EncloseAt(rev, locs)`, with `EncloseResult`, `EncloseReasons` and `RenderEncloseJSON`; paths are relative to the repo root, not the cwd, and an unknown rev fails the whole call.
- They take location spellings (`path:N`, `path:N-M`, `path:N:col`), not structs; Loomyard formats tool-output locations into them.
- Rejections carry a closed reason set (`EncloseReasons`), including `unaddressable` for a `.go` file in the repo root or a path containing `#`; Loomyard cites `path:line` for any rejected item.
- A `found` glyph may not Resolve afterwards (an explicitly named gitignored file, a file without a package clause, a path through a symlinked directory; `quarry/enclose.go`). Loomyard treats a Resolve miss after Enclose as "no current span", never as an error, the same as a member the fixer deleted or renamed.
- Loomyard pins quarry v0.2.1; this task bumps `go.mod` to v0.3.0, which has no other breaking change.
