# Support log — plan-split-producers

## Interview

Autonomous run: no operator answered, and the step had no advisors.
Every question below was settled by the writer and is recorded in the question ledger as an auto-pick.

### Turn 1 — exploration

Read the board entry, its note `plan-writer-context`, the board list and `lym loom peers` (one peer, `gate-fewer-binaries-trial`, touching nothing here).
Read `internal/seatengine`'s package doc and table, prompt and notice code, the Plan-Write and MultiLLM registry entries, `loomengine.PlanTable`, the tapestry recipe and config template, the entry-gate code in `internal/shedbuild` and `internal/shedengine`, the plan spec's layout, numbering, handle and check sections, the seat directives, `validate-plan`'s help and `lym quarry`'s help.

Findings that settled questions without asking:

- `entry_gate` supports `==` and `!=` against a setting, and a goto bypasses the gate, so the swap needs no new mechanism.
- `Required` and `Accept` on a seat have no production user, so their notice rule can change for writer seats.
- A required advisor queues no line on done, so a chair would never be woken by a finished part; the engine needs a done line.
- An advisor's output existing marks it done, so a stale output from an earlier attempt must be archived before a fresh spawn.
- The plan parser takes a directory, so a part can be a plan directory of its own and be checked by the existing check set.
- `planparser.InsertCard` already renumbers files, headings, commit prefixes and card ids.

### Turn 2 — design batch (auto-picked)

1. Switch: entry gates on a `tapestry.plan_split` flag, new rows ahead of `Plan-Write` (D1).
2. Parts file place and format: `_lym/parts/parts.md`, a sole parser, a mechanical check (D3).
3. Parts-Write review: none, as the board's default (D4).
4. Chair as agent, or Go join: both — Go numbers and orders, the agent judges contracts and shared spots (D8, D9).
5. Models: Opus medium for all three, chair on a key of its own (D2).
6. Part validation before the join: a per-seat `plan-part` gate plus a required writer seat (D5, D7).
7. Reading depth: two stencil blocks chosen by `parts_depth` (D4).

### Turn 3 — lens review

All six forks reported; reports are saved beside this log. Duplicates merged.

- interactions 1, existing-paths 1 (BLOCKING): a resume ends the step once any writer finished — fixed: D5 counts a writer whose outputs exist and pass Accept as finished on resume; D6 bound and D5 acceptance updated.
- existing-paths 2: resumed chair told a finished writer no longer answers — fixed: D5 states the line, D9 has the chair fix that part itself.
- existing-paths 3, bound-breaker 3: commit pathspec misses webster's directory and the amendment record — fixed in D6 and its acceptance.
- existing-paths 4: settled-verdict refusal names `Plan-Write` in split runs — fixed in D1: it names the writer the run's history ran.
- existing-paths 5, interactions 5, bound-breaker 7: archive placement and nesting — fixed: D4 and D6 move entries into `_lym/parts/archive-<stamp>/`, never earlier archives.
- interactions 4, bound-breaker 1 and 2: D1's bound claims were false — fixed: D1 states per-row gate reads and a target-only goto bypass.
- interactions 2, walkthrough 3: parts cannot use names a shared spot creates — fixed: shared spots carry exports (D3), counted by the part check (D7), created by the chair (D9).
- interactions 3, walkthrough 2: area/shared-spot refusal too broad — fixed: D3 refuses only an exact file listed by both.
- interactions 6: a fixed part wakes no chair — fixed in D11: the writer replies once the recheck passes.
- interactions 7: rejoin after shared-spot cards discards them — fixed in D9: join once, later seams fixed in the joined plan.
- bound-breaker 4: writers would notify the parent on held turns — fixed in D5: writers hold quietly.
- bound-breaker 5: non-exported handle or path created by two parts — fixed in D8: the join refuses any such collision.
- bound-breaker 6: decision ids would need a second discussion parser — fixed in D3: read through a new `internal/discussionparser` export.
- walkthrough 1: part writers and chair lack the quarry glyph rule — fixed in D12: the quarry subsection moves into the block, which names no step.
- walkthrough 4: chair inputs unstated — fixed in D6.
- walkthrough 5: overlap with the producer's own output archive — fixed in D6's wording.
- walkthrough 6: part overviews would carry `## PATTERN changes` — fixed in D11.
- board-scope 1 (class decision): stencils not settled with the operator — recorded under Open risks for the parent.
- board-scope 2: `medium` depth may lack signatures — fixed: checked `lym quarry toc` and `expand` output in this worktree; D4 now uses `expand` for signatures.
- leanness Scope In list — fixed: In points at the decisions.
- leanness Auto-mode assumptions restating D2 — fixed: one pointer line to the ledger.
- leanness D5 rationale evidence — fixed: moved to a Constraints line.
- leanness D9 rationale clause — fixed: cut, kept in Rejected alternatives and Q10.
- leanness Notes bullets 2 and 3 — fixed: folded into D6.
- leanness Open risks crash bullet — fixed: dropped, D6's bound holds it.
- leanness Goal duplicate clause — fixed: cut from Goal.
- leanness D10 and D11 — no finding by the lens itself.

## Rejected alternatives

- **A branch inside `Plan-Write`'s producer instead of new rows** — rejected: the board asks for two producers beside `Plan-Write`, and separate rows give the bench separate history entries and spans.
- **New rows after `Plan-Write`** — works with the same gates, but changes `Plan-Write`'s `on_done`; ahead of it, `Plan-Write`'s edges stay as they are.
- **A new loom recipe** — ruled out by the board.
- **The chair renumbers by hand** — error-prone across many cards and costs chair turns; the renumbering exists in Go.
- **No chair agent, Go joining everything and a seat writing the shared spots** — the board's open alternative; rejected because judging delivered behaviour against a contract and placing shared-spot cards need judgement Go cannot give.
- **The plan overview as the chair's output** — the join writes the overview, which would mark the chair done before it settled the shared spots.
- **Parts and part cards under `_lym/plan/`** — the plan directory's rotation, Plan-Review and Webster read it; a separate `_lym/parts/` keeps the plan directory an ordinary plan.
- **Parts in `.lym` (untracked)** — the bench needs the parts file and part cards after landing.
- **Full plan check set per part** — `caller-uncovered` and `pattern-changes-mismatch` would flag what another part or the chair covers, and a writer could not fix them, exhausting its gate.
- **A required advisor with only an Accept hook, no seat gate** — a failed Accept ends the step at once; a seat gate re-prompts the writer in session first.
- **A heavy reading depth** — ruled out by the board.
- **Twill lens forks in the chair** — not asked for; Plan-Review reviews the whole plan.
- **Sonnet as the chair's default** — a first bench should change the split, not the model; the key lets the bench try it.
- **Mixed advisor and writer tables** — no user; refusing them keeps each directive's wording simple.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): how the flag swaps the rows — entry gates on `tapestry.plan_split`, new rows ahead of `Plan-Write`. Rationale: existing mechanism, `Plan-Write`'s edges unchanged.
- Q2 (resolved, auto-pick): name and place of the parts file and part cards — `_lym/parts/parts.md`, `_lym/parts/<part>/`, tracked. Rationale: kept out of the plan directory, durable for the bench.
- Q3 (resolved, auto-pick): a review of the parts file — none. Rationale: board default; Plan-Review catches a bad split's effects.
- Q4 (resolved, auto-pick): models — Opus medium for `Parts-Write`, part writers and chair, each on its own key. Rationale: matches `Plan-Write`, bench can vary the chair.
- Q5 (resolved, auto-pick): chair agent versus Go join — Go numbers and orders through `lym loom join-parts`; the chair judges contracts and writes shared spots. Rationale: mechanical work in Go, judgement in the agent.
- Q6 (resolved, auto-pick): catching a failing part before the join — a per-seat `plan-part` gate on a required writer seat, and the join rechecking every part. Rationale: in-session re-prompt first, then the step ends.
- Q7 (resolved, auto-pick): what a `Parts-Write` that cannot split does — writes one whole-task part with its reason. Rationale: keeps the run moving and leaves a countable trace.
- Q8 (resolved, auto-pick): cap on the number of parts — `parts_max`, default 6. Rationale: bounds the number of Opus sessions a split starts.
- Q9 (resolved, auto-pick): lens forks in the new producers — none. Rationale: not asked for.
- Q10 (resolved, auto-pick): what the chair writes as its output — `_lym/parts/join.md`, last. Rationale: an overview output would read as done when the join writes it.
