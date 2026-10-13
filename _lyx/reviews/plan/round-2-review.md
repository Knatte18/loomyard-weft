---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/08-circling-continue-hard-cap.md
    summary: >-
      a loom.yaml hard-cap load refusal reached through circling continue carries two conflicting ways forward, lyx loom resume from card 6 and re-run the verb from circlingWayHardCap
    fix: >-
      In card 8's Intent, make the hardCap-error refusal name the circling verb explicitly in place of card 6's resume step. Change circlingWayHardCap's planned text (its Create signature and the circlingVerb bullet) to `way forward: fix the file or key the message names, then re-run "lyx loom circling continue <slug>" in place of any "lyx loom resume" it names`, with the slug filled from the verb's slug as the hard-cap row's text is. Change the refusal-spec "circling: hard cap unresolved" bullet's way forward to match. Add the phrase "in place of any" to the "hard cap unresolved" row's wantIn in TestCirclingVerb_Refusals.
---

### [LOW] Circling continue's hard-cap-unresolved refusal stacks two ways forward

**Location:** `_lyx/plan/08-circling-continue-hard-cap.md`, the `circlingVerb` bullets and `perchHardCap`.

**Issue:** `perchHardCap` loads loom.yaml through `loomengine.LoadConfig`.
Card 6 makes that function's three new hard-cap refusals end with `then run "lyx loom resume"`.
`circlingVerb` wraps any `hardCap` error as "%s; %s" with `circlingWayHardCap`, so a hub whose loom config was edited into a hard-below-soft state after the run escalated gives the agent `... then run "lyx loom resume"; way forward: fix the file or key the message names, then re-run the verb`.
The two steps disagree.
Running `lyx loom resume` re-steps the Bouncer at the escalation, and the `continue` the caller came to record is lost.
D5 settles the same conflict for the hard-cap refusal itself by putting `lyx loom circling continue <slug>` where D3 has `lyx loom resume`, and this refusal should follow the same rule.
The class is right: a loom.yaml or darn.yaml load failure is a correctness halt under `refusal-spec.md`'s class rule.
The defect is only the conflicting text.

## Focus verdicts

- Card 6 (`06-loom-hard-cap-config.md`): no defect.
  Every new `LoadConfig` refusal goes through `CapRoute`, never through the static "in loom.yaml" text: `review_hard_max_bounces` below 1, each per-perch hard key below 0, and hard below soft (which names both keys, both values and both routes).
  `CapRoute` picks the route from `configengine.StackLayers`' `Origins`, the highest layer holding the key, which is the layer a fix has to change; `layers.go:307` confirms that shape.
  A key absent from every file layer falls to the template case and the default-layer route.
  The local route carries `lyx board upsert` and "in the task worktree".
  The default route carries "from the hub's prime worktree", as the added Discussion decision requires.
  Each message ends `then run "lyx loom resume"`.
  The new refusal-spec row lists all three commands, the prime, `lyx board upsert` and `lyx loom resume`.
  The `TestLoadConfig_Refuses` plan reaches the worktree route through `writeLoomConfigWithKeys`, which seeds the full template, so the hard key's origin is the worktree layer.
  It reaches the local and prime routes through the hub-stack subtest, with resume asserted for each refusal kind at least once.
  The new keys are classified `none`, which `ValidateLocal` admits, so `lyx config loom --layer local --set` can write them.
  None of the named commands is on `PATTERN-no-denied-recovery`'s denied list.
- Card 8 (`08-circling-continue-hard-cap.md`): the classes are right, and the only defect is F1's text.
  "circling: continue at the hard cap" is a correctness halt.
  A `latestRound` failure is a stat failure in `ResolveRound` (`round.go:27`), and it joins the section's existing transient row "circling: decision record failed".
  "circling: hard cap unresolved" covers loom.yaml and darn.yaml load failures as a correctness halt, as the class rule requires.
  `perchHardCap`'s unknown-row error is a wiring guard, which the loom section's grouped "wiring guards" row covers with no per-row test.
  `cmd/lyx/testdata/test-budget.yaml`'s 221 for `internal/loomcli` equals today's count of top-level tests without `TestMain`, so 222 is right for `TestPerchHardCap`.
