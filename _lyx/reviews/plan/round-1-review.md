---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/06-loom-hard-cap-config.md
    summary: >-
      Card 6 gives the loom review_hard_max_bounces-below-1 and negative per-perch hard-cap refusals the static in-loom.yaml way forward, not D3's per-layer route and lyx loom resume, so a run_settings value gets a way forward that cannot clear it
    fix: >-
      In card 6's LoadConfig bullets, route both new refusals through CapRoute. For review_hard_max_bounces below 1, the message names CapRoute(dirs, "review_hard_max_bounces"). For each per-perch hard key below 0, it names CapRoute(dirs, <that key>). Both messages end with then run "lyx loom resume". Do not reuse the existing at-least-1 loop's "set it to a positive integer in loom.yaml" text or the per-perch "..., in loom.yaml" text for the hard keys. In the refusal-spec bullet, give these triggers the per-layer way forward the new hard-below-soft row carries, instead of only widening the "config: review key below 1" row's static way forward. In TestLoadConfig_Refuses, add a row inside the hub-stack subtest: a local review_hard_max_bounces of 0 wants lyx config loom --layer local --set review_hard_max_bounces=<n>, lyx board upsert and lyx loom resume. The other review_hard_max_bounces-0 row and the per-perch -1 rows want the route of their layer and lyx loom resume. No target changes, since LoadConfig, refusal-spec.md and TestLoadConfig_Refuses are already card 6 targets.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/08-circling-continue-hard-cap.md
    summary: >-
      Card 8's new refusal-spec row "circling: hard cap unresolved" classes an unreadable run directory (a raw stat failure from LatestRound) as a correctness halt, while refusal-spec.md puts raw I/O failures on a transient row
    fix: >-
      In card 8's refusal-spec bullet, scope "circling: hard cap unresolved" to loom.yaml or darn.yaml failing to load, which stays a correctness halt. Move the latestRound read failure onto the circling section's existing transient row, "circling: decision record failed". Widen that row's trigger to "or cannot read the run directory's latest round for the hard-cap check"; its way forward ("re-run the verb; if the failure persists, fix the file or directory the message names") stays. circlingVerb's latestErr branch then refuses with circlingWayRetry instead of circlingWayHardCap. The "latest round unreadable" row of TestCirclingVerb_Refusals wants "re-run the verb".
---

### [BLOCKING] Loom hard-cap below-1 and negative refusals lack D3's per-layer way forward

**Location:** `_lyx/plan/06-loom-hard-cap-config.md`, the `LoadConfig` bullets and the `refusal-spec.md` bullets.

**Issue:** D3 says: "Each refusal's way forward is to raise the hard cap or lower the soft cap in the layer the resolved key comes from, written so it reaches the task worktree's own config, then `lyx loom resume`".
It then lists the local (`run_settings`), worktree, hub-default-from-the-prime and darn routes.
Focus item 2 names D3's refusals as three: hard below 1, per-perch negative, and hard below soft.
Card 6 applies the per-layer route (`CapRoute`) only to hard below soft.
For `review_hard_max_bounces` below 1, it joins "the at-least-1 loop beside `review_max_bounces`, with its way forward".
That way forward is the existing text at `internal/loomengine/config.go:903`, "set it to a positive integer in loom.yaml".
A negative per-perch hard key gets "set it to 0 to take review_hard_max_bounces, or to a positive integer, in loom.yaml", copied from `config.go:918`.
Neither text names `lyx loom resume`.
The refusal-spec change only widens the "config: review key below 1" row's trigger and its static way forward.

This matters because `ConfigSpec` classifies all five new keys `none`, so they can be set through `run_settings`, which Seed-Child writes into the local layer.
Take a task whose `run_settings` sets `review_hard_max_bounces: 0`.
"Set it in loom.yaml" sends the agent to the tracked file, but the local layer still overrides it, so the refusal comes back.
Without the `lyx board upsert` half, the next Seed-Child writes the bad value again.
That is the defect the decision record's carried-over Discussion finding `hard-cap-way-forward-misses-run-settings-layer` was about.
It also breaks the `PATTERN-refusal-way-forward` constraint the decision record lists, because each new refusal must name a way forward that works.
The darn side (card 7) has only the hub route, so it is not affected.

### [LOW] An unreadable run directory is classed as a correctness halt

**Location:** `_lyx/plan/08-circling-continue-hard-cap.md`, the `refusal-spec.md` bullet "circling: hard cap unresolved".

**Issue:** The row's trigger covers two failures: "cannot read the run directory's latest round" and "loom.yaml or darn.yaml fails to load".
It classes both as a correctness halt.
`LatestRound` resolves through `ResolveRound`, whose failures are `stat` errors on the run directory or a report file (`internal/shedadapters/round.go:27-48`).
`contracts/specs/refusal-spec.md:33` says "Raw I/O failures (`stat`, `mkdir`, `write`) form one grouped transient row per section".
The circling section already has a transient row for its own file failures, "circling: decision record failed" (`refusal-spec.md:285`).
A config load failure is a correctness halt, but a stat failure is transient, so one row should not carry both.

## Rejected

- Card order (focus item 5): `EscalationHardCap` comes from card 2, `HardCap`, `atHardCap` and the fragments from card 3, `RowHardMaxBounces` from card 4, and `Caps`/`CapRoute` from card 6. Each is used only by itself or by later cards (card 7's darn key is used by card 8), so no card uses a later card's symbol.
- Verify coverage (focus item 6): `go test ./...` and `go test -tags integration ./...` cover every targeted package (shedadapters, shedrecipe, testkit/envkit, loomrecipe, loomengine, loomcli, configreg, darnengine, contracts/stencils, cmd/lyx). The section follows the template order: build, vet under integration, tmux and llm, untagged tests, then integration tests.
- The unreadable-round-ledger branch of card 5's step 1 has no test row: `readJudgment` (`internal/shedadapters/bouncerfiles.go:141-170`) already requires that ledger to parse before `settle` is reached. The card says this and keeps the branch as a guard, so this is not a defect.
- Card 5's previous-decision read error escalates with cause `budget`: this is licensed by the Goal's "where the rule cannot read its inputs", and step 4 cannot apply D2's "after a continue" test without reading that file.
- Test budget: shedadapters has 154 top-level tests against a row of 155, so card 5's one new test fits. loomcli has 221 against 221, and card 8 raises the row for `TestPerchHardCap`. loomrecipe deletes two tests and creates two.
- Card 5's rewritten rows in `TestBouncer_AutoAccept` and `TestBouncerEntry_BlockingEscalatesAndColorSegment` do reach the rule's continue. The Bouncer's report (`round-N-report.md`, written with `bouncerReport`) and the shedrecipe fixture's `round-1-review.md` both parse. The unparseable `review\n` lands only at `roundReviewPath`, which the rule's step 1 does not read.
- Card 3's retired names (`round-2-escalation-<stamp>.md` and the like, written by `archiveStaleOutputs`) are ignored by `readEscalation`, `readCirclingDecision` and `ReviewRounds`, all of which match exact file names.
- Card 2's ImpactSummary is accurate: nothing outside `internal/shedadapters` names `EscalationBudget` or `EscalationCircling`.
- Stencil constraints (focus item 10): the judge stencil gains no marker and no `{{if}}`/`{{range}}`. The decide fragments follow the existing `webster-fragment-*` precedent. The brief and the arbiter stencil point at the judge stencil's meanings without restating them. Card 9 carries `review_hard_max_bounces: 3`.
- Scope (focus item 9): no card touches `arbitereligibility.go`, the driver stencil, `lyx report convergence` or the meaning of `*_auto_accept`.
- Attack surface (focus item 4): card 3 bounds the exempt continue by the hard cap in the same sentence that introduces it. Card 5 carries D2's Bound paragraph. Card 8 adds a guard and removes none.
