```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the code paths that already write or read what each decision adds or changes, and checked for each decision whether its result survives them.

**Findings**

1. **BLOCKING — D1 and D8: on a sampler run, Describe can be skipped and leave no change description.**
   - D1 keeps darn's `entry_gate: budget.Code-Review` on Describe.
   - In darn that gate skips Describe whenever the Code-Review budget (`loom.code_max_bounces`) is 0, because Darn-Write has already written its own description. The darn recipe's comment says "a budget of 0 turns the perch off: the Bouncer settles approved at once and Describe's entry gate skips it, so the writer's own …" (`contracts/recipes/darn-recipe.yaml`, lines 37-39 and 57-58).
   - Under D8 the sampler agent writes no description, and Finalize fails without one: `summaryparser.Parse(fz.deps.DescriptionPath)` returns the error "parse change description" (`internal/landingshed/finalize.go:185-187`).
   - D2 gives the `speed` profile a sampler block equal to its darn block, which sets `code_max_bounces: 0`. So a sampler task naming `speed` skips Describe and breaks at Finalize. A hub whose `loom.yaml` sets 0 does the same.
   - D3 also splits "perch off" across two keys, `sampler.code_review` and `budget.Code-Review`, and Describe reads only the second.
   - Fix: on a sampler run Describe always runs, either with no `entry_gate` or gated on something the agent cannot leave empty. The acceptance criteria should cover a budget-0 profile.

2. **MEDIUM — D1: the tree-scope wiring is keyed on the `DarnWrite` engine, which sampler lacks.**
   - `wireDarn` runs only when the recipe has a `DarnWrite` row (`internal/loomcli/wiring.go:869-878`, through `loomrecipe.HasEngine(c.recipe, darnWriteEngine)`).
   - It fills `env.VerifyTreeScope`, `env.VerifyMergeBase`, `env.DarnVerifyAttempts`, `env.SegmentBounces` and `env.VerifyCommand` (`internal/loomcli/darnwiring.go:210-218`).
   - With `code_review` on, sampler's `Code-Burler-Fix` verify gate would get no tree scope. It would fall back to the key's whole scope (`internal/shedrecipe/entries_gate.go:200`) and lose darn's `verify_attempts` budget. The bouncer would also get no `SegmentBounces` reading (`entries_bouncer.go:265`).
   - Finalize's own verify source is fine: `arm.go:345` reads `verify_source`, so D1's claim holds for Finalize only.
   - The record says D1 reads tree scope "as darn's verify sites do" but never says this wiring must stop keying on `DarnWrite`. It should say these seams follow `verify_source: tree_scope` or the perch rows, never the `DarnWrite` engine.

3. **MEDIUM — D6: the record check D6 calls "the check Darn-Write's post-session guard runs" is also wired only for a `DarnWrite` recipe.**
   - `Darn.CheckRecord` and `Darn.CommittedRecord` come from `darnDeps` inside `wireDarn` (`darnwiring.go:218`). A sampler run gets neither unless the wiring changes.
   - The record does not say how sampler gets these seams.

4. **LOW — D4: decision add's way-forward texts name rows that do not fit a sampler run.**
   - The record checks in `checkDarnRecord` and `missingRecordRefusal` name ways forward by the current row (`internal/loomcli/decision.go:392-450`).
   - For a sampler run at `Sampler-Session` they fall to the past-Record-Write branches: restore, or goto. That reads correctly.
   - But D4's new refusal for "that strand once the row has ended" names "the run's parent" as the way forward. A sampler agent has already been stopped by then (D6 stops the session at Done), so the refusal can never reach it. It is harmless and fits the bound.

5. **LOW — D7: the shared list's home is constrained by an existing import.**
   - Dobby's reading in `battencli.childAtWaitingRow` imports `pawlengine.NameBeamAwait` and `loomshed.NamePRReview`.
   - The watchdog reading lives in `loomcli`, so the list must sit where both can import it, not in `battencli`.
   - The record leaves this to the plan, which is fine. Noted only because "one list" depends on it.

**Paths traced, per decision**

- **D1:** Describe's `budget.Code-Review` gate breaks (finding 1). Finalize's `verify_source` survives (`arm.go:345`). `Code-Burler-Fix`'s gate tree scope breaks (finding 2). Record-Write's deps are wired unconditionally at `wiring.go:698` and survive.
- **D2:** I did not trace the `runprofile` `Settings` block refusal further; it is already in context. The speed profile's sampler block feeds finding 1.
- **D3:** The perch-off switch and Describe's gate disagree (finding 1).
- **D4:** The `checkDarnRecord` and `missingRecordRefusal` row branches survive for a sampler run (finding 4).
- **D6:** The record-check seams are only filled for `DarnWrite` recipes, so this breaks (finding 3).
- **D7:** The `childAtWaitingRow` and `agentActivityFor` readers survive, with the import constraint in finding 5.

**Tool calls**

1. Grepped `darn-recipe.yaml` for Describe and budget, and `finalize.go` for the description parse: my context lacked what Describe's `entry_gate` means and whether Finalize needs a description.
2. Grepped `loomcli` for row-name and `DarnWrite` branches: my context lacked which wiring keys on darn's rows.
3. Read `decision.go:386-470` and `wiring.go:860-900`: my context lacked the record-refusal row switch and the `wireDarn` gate.
4. Grepped the Record-Write entry, the `tree_scope` wiring and the `wireDarn` body: my context lacked which seams `wireDarn` fills.
5. Grepped `shedrecipe` readers of `VerifyTreeScope`, `DarnVerifyAttempts` and `SegmentBounces`: my context lacked whether those seams are optional and what a nil one does.
6. Grepped where `RecordWrite` and `VerifyCommand` are wired: my context lacked whether Record-Write's deps depend on `wireDarn`.
