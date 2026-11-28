---
verdict: BLOCKING
findings:
  - id: F13
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:117
    summary: "D5's changed lines hand a non-batten jacquard pair whose entry lost its record to Record-Seed's Stuck, but D4's Stuck reason (line 80) names only the batten sequence, which cannot clear a run batten does not hold, against PATTERN-refusal-way-forward"
    fix: "In D4's way-forward list at `_lym/discussion/decision-record.md:80`, make the jacquard bullet name two cases, as D5's `lym loom start` refusal does at lines 113-115. A pair batten made keeps the current sequence: `lym batten cancel <slug> --remove`, `lym board upsert '{\"slug\":\"<slug>\",\"recipe\":\"tapestry\"}'`, `lym board set-status '{\"slug\":\"<slug>\",\"status\":null}'`, then `lym batten run <slug>`. Any other pair gets its own way forward that ends the jacquard run and restarts the work as tapestry. The writer must establish that way forward in the code and cite its evidence. Unverified: no `lym loom` verb that switches a started run's recipe was read, and a seed rewrite alone may not clear the Stuck, because the status file already exists (`internal/loomcli/sharedbootstrap.go:128-131`). In the D4 acceptance criterion at line 236, replace `a reason naming both ways forward` with a reason per case: the tapestry goto, the jacquard batten sequence, and the jacquard non-batten way forward. Keep it tier 0 in `internal/loomshed`."
  - id: F14
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:123
    summary: "D5 claims the motif-run clear leaves the motif run's task branch, and the Notes (lines 259-261) leave the plan writer to confirm whether a jacquard start meets it, with a conditional ClearDoneRun fallback. The batten-site `lym batten cleanup --apply` way forward (line 108) rests on the same unsettled fact."
    fix: "Settle the fact and record it with its evidence. Evidence read: `internal/fabricengine/remove.go:76-77` has Remove delete the local task branch through the destructive gate unless its work is on no other ref, `:89-90` has it delete the origin branch through the landed-work gate, and `internal/battencli/wire.go` `teardownRequest` passes `Remote: true`. A motif run lands no code (`contracts/recipes/motif-recipe.yaml:8`). Unverified: whether a motif run's warp task branch ever holds a commit beyond its parent. Confirm that from batten's Worktree-Teardown (`internal/battenshed/teardown.go`). If it holds none, replace line 123's second sentence with: `A motif run commits no work to its task branch, so its teardown deletes the branch locally and on origin, and the start that follows the clear meets no leftover branch.` Cite remove.go in that sentence, add the same statement as a Fact under `## Constraints`, and delete the second Notes bullet (lines 259-261). If it can hold one, the jacquard clear goes through `ClearDoneRun`, which archives and deletes the branch (`internal/battencli/cleanup.go:157-168`). The batten-site way forward at line 108 then names `lym batten cancel <slug> --remove` from the hub's prime worktree in place of `lym batten cleanup --apply`, since Cleanup touches no branch (`cleanup.go:111-114`)."
  - id: F15
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:146
    summary: "D7's reinterpretation derives a jacquard block from the tapestry block only for an operator-added profile. On a hub that tuned a shipped profile's tapestry block or `defaults.tapestry`, the follow gains the template's jacquard values, so raddle-filed parts that ran the tuned values silently run shipped ones. That meaning change is left without the reinterpretation PATTERN-config-layers requires."
    fix: "In D7 at `_lym/discussion/decision-record.md:146-151`, widen the one-time reinterpretation's second case. Every profile that holds a tapestry block gets a jacquard block equal to that tapestry block minus `loom.discussion`. For a shipped profile left as shipped, that equals the template's block, so the rewrite changes nothing. Add a case: `defaults.jacquard` is set to the profile `defaults.tapestry` names when that profile then holds a jacquard block, and is dropped otherwise. That case subsumes the existing drop of a dangling default. State that the reinterpretation runs after the template follow in the same reconcile walk (`internal/hubreconcile/walk.go:250-266`). State also that a derived block whose values differ from `profiles.template-base` is kept as the operator's on later reconciles (`internal/yamlengine/follow.go:167-171`). Extend the D7 acceptance criterion at lines 248-249 (tier 0, `internal/runprofile`). A shipped profile whose tapestry block the operator changed gains a jacquard block carrying the changed values, and a changed `defaults.tapestry` carries into `defaults.jacquard`."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
    answer: "D5 now has `startClearsMotifRun` admit a start whose recipe needs a stored record, with a tier 1 criterion. The leftover task branch that clear meets is new finding F14."
  - id: F4
    status: resolved
    answer: "D7's reinterpretation covers both named cases, and the reinterpretation runs after the follow (`internal/hubreconcile/walk.go:250-266`). The tuned shipped profile is new finding F15."
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: resolved
    answer: "Accepted the different fix. The check stats the status file before `loomshed.Seed` and runs only when the file is absent, so a re-run over an existing status file passes, as the criterion now tests."
  - id: F8
    status: resolved
    answer: "Settled by the added entry `Jacquard stays one recipe of every row after Discussion; the plan-and-build-recipes note is later work`."
  - id: F9
    status: resolved
    answer: "Settled by the added entry `The jacq nickname is written by the run's parent after landing, as D11 says`, and again by `Nickname jacq written after landing`."
  - id: F10
    status: resolved
  - id: F11
    status: resolved
    answer: "Accepted the different fix. The rationale names every check other than the worktree and fabric checks as one that guards only build-row config."
  - id: F12
    status: resolved
---

This round is incremental: the round-1 fixes changed `_lym/discussion/decision-record.md` alone.
All three new findings sit on hunks those fixes added (D5's way-forward and clear paragraphs, and D7's reinterpretation).

### [BLOCKING] Record-Seed's jacquard Stuck has no way forward for a pair batten did not make

**Location:** `_lym/discussion/decision-record.md:117`, fix site D4 at line 80

**Issue:** D5 (lines 113-118) now separates a batten-made pair from any other, and hands a re-run over an existing status file to Record-Seed's Stuck.
That Stuck's jacquard reason, D4 line 80, names only `lym batten cancel <slug> --remove` … `lym batten run <slug>`.
In a worktree batten did not make, batten holds no run to cancel, so following the way forward leaves the run Stuck.
`PATTERN-refusal-way-forward` requires every refusal to name a way forward that clears it.
The D4 criterion's "a reason naming both ways forward" also conflicts with D4's one-reason-per-recipe text.
Raised by the walkthrough and interactions lenses.

### [BLOCKING] The motif-run clear's task-branch fact is deferred to the plan writer

**Location:** `_lym/discussion/decision-record.md:123`, with the Notes at lines 259-261 and the way forward at line 108

**Issue:** D5 says the clear "leaves the motif run's task branch" and that a jacquard start meets it as a raddle start does.
The Notes then call that claim unverified and give the plan writer a conditional switch to `ClearDoneRun`.
The rubric counts this as a deferred research question.
The batten-site way forward for a slug that holds a done motif run (`lym batten cleanup --apply`) rests on the same fact, because `Cleanup` touches no branch and Worktree-Create halts on a leftover one (`internal/battencli/wire.go:380-394`).
The code read points to the fact holding: Remove deletes a task branch whose work is on another ref, locally and with `Remote: true` on origin (`internal/fabricengine/remove.go:76-90`).
That still has to be confirmed for a motif pair and written into the record.
Raised by the bound-breaker and walkthrough lenses.

### [BLOCKING] D7 drops an operator's tuned tapestry values for the runs that become jacquard

**Location:** `_lym/discussion/decision-record.md:146`

**Issue:** A key that both the file and `profiles.template-base` lack takes the template's value (`internal/yamlengine/follow.go:173-181`).
So every shipped profile gains the template's jacquard block, and `defaults.jacquard: solid`, whatever the operator set under its tapestry block or `defaults.tapestry`.
After D6, raddle-filed parts run jacquard and read that block, so a hub's tuned review budgets stop applying to them without notice.
`PATTERN-config-layers` has a meaning change ship a reinterpretation, and D7's reinterpretation covers only operator-added profiles.
Raised by the existing-paths and bound-breaker lenses.

## Rejected

- `lym loom step` seeds a first status file through `seedAndCommitBootstrap` and so passes over D5's start check (three lenses, LOW): an incremental round raises only BLOCKING findings, and Record-Seed's Stuck still stops such a run before any build row.
- The D1/D2 criterion's "same edges" against block-exit rewriting (interactions, LOW): below the incremental bar. The criterion compares the expanded recipe, whose inlined rows are what a test reads.
- The D4 criterion's "a reason naming both ways forward" (interactions, LOW): folded into F13's fix.
- The D11 Open risks line is now settled by two added entries: the line was not changed this round, the finding would be below the bar, and the arbiter's entry orders the line kept.

## Focus verdicts

- Start refusal sites: batten's site and `lym loom start` each name a way forward that clears the refusal, and the D5 criterion has a test reach both. The Stuck those sites hand over to is F13.
- `lym loom start` check placement: a first start is refused before `loomshed.Seed` writes a status file. A re-run over an existing status file passes to Record-Seed. Not a defect.
- Decision verbs: `recordWritePending` reads Preflight, a row jacquard holds, and D4 has every Record-Write literal read through `loomrecipe.EngineRow`. Not a defect.
- Jacquard over a done motif run: the clear is reachable, and the task-branch outcome is F14.
- profiles.yaml reconcile: the file still loads, since the reinterpretation runs after the follow. The tuned-profile gap is F15.
- Empty `on_done` on the last block row: the `always` gate never skips, so `skipTarget`'s fallback to the block entry is unreachable, and D3's bound holds. Not a defect.
