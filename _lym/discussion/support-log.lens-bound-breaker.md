```yaml
lens: bound-breaker
outcome: reported
```

Task: try to break each bound, guarantee and claim about existing code in the record (bound-breaker lens).

**Findings**

1. **MEDIUM — D1 bound ("the gate is read once, on entry; a flag changed after `Drift-Baseline` does not move it") is false.**
   Each row's gate is evaluated when the run enters that row (`internal/shedengine/entry.go`; `EnteredRow` is per row).
   Counterexample: `plan_split` is true when the run enters `Parts-Write`. The flag is set false in the local layer while `Parts-Plan-Write` runs. `Parts-Plan-Write`'s Done then enters `Plan-Write`, whose `!= true` gate now holds, so `Plan-Write` runs and its rotator archives the joined plan. The reverse flip, between `Parts-Write` and `Parts-Plan-Write`, skips `Parts-Plan-Write` and runs `Plan-Write` after a finished parts file.
   The bound needs to name this, or the gates need to read one value fixed at `Parts-Write` entry.

2. **MEDIUM — D1 bound ("a goto onto either branch runs that branch's rows whatever the flag says") is broken.**
   A goto bypasses only its target row's gate (`goto.go` sets `EnteredRow = req.Target`). The rows after it evaluate their gates as usual.
   Counterexample: in a run with the flag off, `lym loom goto --to Parts-Write` runs `Parts-Write`. On Done, `Parts-Plan-Write`'s gate is false, so it is skipped. `Plan-Write`'s gate holds, so a solo plan is written and the parts file is silently ignored.
   Likewise, `goto --to Parts-Plan-Write` with the flag off runs the chair, then `Plan-Write`, which overwrites the joined plan.
   Only a goto to `Plan-Write` in a split run behaves as claimed.

3. **MEDIUM — D6 commit scope misses webster's directory.**
   D6 reuses `loomshed.NewPlanDirRotator`, which takes `Rework.ArchiveWebster` and moves webster's tracked run record. `planCommitPathspec` (`internal/loomcli/wiring.go:118-139`) exists to stage those deletions, plus the amendment record.
   D6 says the decorator commits "the plan directory and `_lym/parts/`". Counterexample: a goto back to `Parts-Plan-Write` after Webster began leaves the webster deletions unstaged, so the worktree is dirty after Done.
   D6 should commit `planCommitPathspec`'s set plus `_lym/parts/`.

4. **MEDIUM — D5 writer seats will notify the parent, and the record says nothing about it.**
   seatengine doc: "A required advisor's held turn end still notifies the parent, where every other advisor holds quietly." Writers are required (D5).
   Counterexample: three writers that raise contract questions and end their turns waiting for the chair's ruling each send a held-turn notice to `ly:orch`. A writer kept idle after done (D5 "keeps its pane") would too.
   D5 should either state this or exempt writer seats from the parent notice.

5. **MEDIUM — D8/D3 bound: only exports are checked for collisions, so the join can produce a plan that fails its own gate.**
   Two parts can each `Create` the same non-exported `plan:` handle (for example a shared test helper) or the same new file path. Neither part check sees it, and the parts check only refuses an export claimed twice.
   The joined plan then fails `handle-collision` or `path-missing`/create overlap. The chair would have to edit joined cards, but a rerun of `join-parts` discards those edits (D8 bound).
   `join-parts` should refuse cross-part collisions of any created handle or path, so the fix goes back to a part writer.

6. **LOW — D3: the parts check reading the record's decision ids collides with PATTERN-sole-parsers.**
   `internal/discussionparser` exports only `CheckDecisionIDs` (findings), with no list of ids. `partsparser` cannot parse `### D&lt;n&gt;` headings itself without becoming a second discussion parser.
   The record should name a new `discussionparser` export, or route the check through it.

7. **LOW — D4 archive nesting.**
   "Archives a stale `_lym/parts/` into an archive directory beside it": if the archive lands inside `_lym/parts/`, the next fresh spawn archives the earlier archive inside the new one. `NewPlanDirRotator` avoids this by moving files only.
   The record should state where the archive goes, for example `_lym/parts-archive-&lt;stamp&gt;/` or files only.

**Claims attacked**

- D1 skip chain routes off→`Plan-Write`, on→`Plan-Bouncer`: held.
- D1 "gate read once": broken (1).
- D1 goto bound: broken (2).
- D1 no `on_stuck`, Stuck halts: held.
- D2 tapestry keys usable as gate operands (`configreg.CheckSettingKey` wired in loomcli): held.
- D2 `parts_depth` refused on load: not verified, left.
- D3 file order is a topological order, so exporters come before users: held for part cards. Shared-spot ordering is already in Open risks.
- D3 export collisions refused: held, but too narrow (5).
- D3 decision-id coverage: implementable only with a new discussionparser export (6).
- D4 stale-output archive before fresh spawn: held in intent; archive placement unbounded (7).
- D5 "`Required` has no production user": held (grep finds no production `Required`/`Accept` use).
- D5 done line: a real gap today. engine.go:121 says a required advisor's end queues no line. Held as a needed change.
- D5 parent notices: broken (4).
- D6 resume ends the step on a lost writer: held per doc.
- D6 commit scope: broken (3).
- D7 the existing parser reads a part directory (`ParsePlan(dir)`): held.
- D8 renumbering reuses `InsertCard`: held. Plan verbs are refused only on a recipe without a `Plan-Write` row (`arm.go` `refuseVerbOffRecipe`), and tapestry keeps that row, so `insert-card` and `validate-plan --part` stay served.
- D8 refuses on a recipe without the row: held as a new rule, consistent with `refuseVerbOffRecipe`.
- D9 chair output `join.md` avoids an early done: held.
- D12 `Plan-Write` prompt byte-identical: plausible; not testable from context.

**Tool calls**

- Grepped `SettingKnown` wiring and the loomcli insert-card and refusal code: the context lacked how tapestry operands and plan verbs are gated.
- Grepped discussionparser exports and seatengine `engine.go` Required/Accept: the context lacked whether decision ids are exported and how required ends queue lines.
- Read `arm.go` `refuseVerbOffRecipe` and the insert-card wiring: to check whether plan verbs serve the new rows.
- Grepped landingshed/generation and `planCommitPathspec`: to check the commit scope the rotator needs.
