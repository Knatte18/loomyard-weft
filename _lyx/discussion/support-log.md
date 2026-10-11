# Support log — plan-glyph-tuning

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry and the related notes (`kick-start-pack`, `enclose-kickstart`, `weaver`, `fan-master-preread`, `discussion-premise-cites`, `webster-parallel-execution`).
   The two kick-start notes wait on this task's measurement; `weaver` names a per-batch pack as open; nothing else overlaps.
2. Explored at module level: `BeginBatch` re-resolves the plan through `planindex.Index` and renders the fork prompt into a file Master forwards by path; `postBatchChecks` is the terminal pass shared by record-batch and recover-batch and calls `BindHandles`; `webster.yaml` loads through `LoadOrTemplate`, which fills missing template keys; quarry v0.2.1's `Symbol` carries `file`, `start`, `end` and `signature`.
3. Settled the switch (D3), the pack's content (D4), its build point (D5), its fork-only reach (D6), the prompt mechanism (D7), the empty and `language: none` cases (D5, D8) and the bind trace (D2).
4. Added a per-begin trace line (D9) so the post-landing audit can tell pack-on batches from pack-off ones.
5. Corrected three premises while writing: the toggle command is `lyx config webster --set`, the prompts live under `.lyx/webster/prompts/`, and `webster-spec.md` does not describe the fork prompt.
6. Lens review; every fork returned a report. Duplicates are merged and named by every lens that raised them.
   - premises, scenarios, pattern-fit (MEDIUM): a not-done card stops binding for the whole batch, so its done siblings got no line — fixed: `batch-not-done` reason in D2, plus a mixed-batch scenario.
   - premises, scenarios, pattern-fit (MEDIUM): under `ErrQuarryUnavailable` `BindHandles` still runs, which gave two lines per handle — fixed: `delta-unavailable` replaces the computed outcome (D2).
   - scenarios (MEDIUM): a line logged inside `planglyph` cannot carry the batch number — fixed: `BindHandles` returns outcomes and `websterengine` logs every line (D2).
   - board-fit, goal-scope (MEDIUM): the with/without audit has no owner once this task closes — fixed: named under Open risks for the landing report.
   - goal-scope (MEDIUM): D3 did not name the comparison's off side — fixed: same-binary runs with the key `false` (D3).
   - scenarios (LOW): error paths log nothing, which D2 did not say — fixed: stated in D2.
   - scenarios (LOW): no scenario for a bare `no-delta-match` or for `delta-unavailable` — fixed: both added to Acceptance criteria.
   - scenarios (LOW): a mid-run switch flip — fixed: D3 allows a mixed run, and D9 records it.
   - scenarios (LOW): `plan:` refs under labels other than `Create` — fixed: D4 lists every `plan:`-shaped ref.
   - scenarios (LOW): a non-boolean value — fixed: D3 and Acceptance criteria.
   - premises (LOW): the pack could read the in-memory plan, older than the plan on disk — fixed: D4 builds from the plan on disk after re-resolution.
   - premises, pattern-fit (LOW): the new stencils need seed registration — fixed: D7.
   - premises, pattern-fit (LOW): a stale `_lyx` path in the `webster-prefix-fork.md` banner — dropped in round 1: the banner is outside this task's scope.
   - goal-scope (LOW): the Rename and Move old sides, and D9, are unlicensed picks — fixed: listed under Auto-mode assumptions.
   - goal-scope (LOW): the operator's "still read the code" rule was not pinned by any criterion — fixed: added to Acceptance criteria.
   - board-fit (LOW): `weaver` reuses the recovery render — fixed: one line in Scope Out.
   - board-fit (LOW): the Webster pack sets no read budget, unlike the Burler design — rejected: the operator's Webster rule is that the fork reads the code, and `kick-start-pack` owns the Burler difference.
   - leanness (LOW): the Scope pack bullet repeated D3-D9 — fixed: cut to a reference.
   - leanness (LOW): D4 and D7 restated Constraints bullets — fixed: the sentences were cut from D4 and D7.
   - leanness (LOW): two Open risks bullets described one risk — fixed: merged.
   - leanness (LOW): Auto-mode assumptions restated their decisions — fixed: trimmed to the pick and a reference.
   - leanness (LOW): D2 named a package placement — superseded: the scenarios fix makes the placement a requirement (batch number on every line).
   - leanness (LOW): the toggle wording in D3's rationale was loose — fixed.

## Rejected alternatives

- **Append a pack section and leave the quarry locate sentence in place.**
  Rejected: the fork would get two contradicting locate instructions, and the operator ruled the fork does not call quarry itself.
- **Render the locate text from Go strings.**
  Rejected: `PATTERN-stencil-ownership` puts prompt text in stencils read through `stencilstore`.
- **Write the pack to a separate file the fork reads.**
  Rejected: one more read per fork for content that fits in the prompt file Master already never reads.
- **Put the pack into the recovery strand's prompt too.**
  Rejected: the board names the fork as the first user, and mixing recovery strands into the comparison muddies it.
- **Refuse `begin-batch` when the pack's resolve fails.**
  Rejected: the pack is an aid; the batch is already validated by `ValidateDispatch`.
- **Default `kickstart_pack` to `false`.**
  Rejected: no run would measure the pack until someone flips it; `lyx config webster --set kickstart_pack=false` restores today's prompt.
- **A read budget for the fork, as in the Burler pack design.**
  Rejected: the operator ruled the Webster fork still reads the code the spans point to; the pack says where to read and does not limit how much.
- **Resolve self glyphs and plain paths too.**
  Rejected: they name whole files the card already names, so a span adds nothing.
- **A mechanical plan-gate check for test-file paths.**
  Rejected: the board asks for the stencil and the rubric, and a path is still right for a test file with no member change.
- **Log bind outcomes only inside `BindHandles`.**
  Rejected: handles of a not-done card never reach it, so the count would miss them.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): default of `kickstart_pack`? — `true`; see D3.
- Q2 (auto-pick, resolved): does the recovery strand get the pack? — no; see Scope.
- Q3 (auto-pick, resolved): which refs go into the pack? — member glyphs under target labels and `Uses:`; see D4.
- Q4 (auto-pick, resolved): what happens on a whole-call resolve error? — all-unresolved pack plus a Warn line; see D5.
- Q5 (auto-pick, resolved): how does the fork handle spans shifted by its own edits? — find the member by its listed signature in that file; see D7.
- Q6 (auto-pick, resolved): is the with/without audit part of this task? — no, it needs real runs after landing; D9 makes it countable.
- Q7 (auto-pick, resolved): is a pack-size cap needed? — no; see Auto-mode assumptions.
