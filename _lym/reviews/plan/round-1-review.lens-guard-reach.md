```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: per guard, whether a real entry point can reach the state it guards.

**Per guard**
- Card 1, fence rows in `TestAppendDecision_*` and `TestValidate_MissingHeadings`: reachable. Each row is a direct input to `fencedLines` and `missingSections`.
- Card 2, `TestParseSeed` beam row and `TestValidateRecipe` beam row: reachable.
- Card 3, `Parse` refusing an unknown `contract` or `verify_source` value: reachable. Parse decodes with `KnownFields` and then runs the shape checks.
- Card 3, `checkLoomContract` clauses 1–5: reachable. `Build` calls it first, before any registry lookup. Parse's shape checks (entry, terminals, producers, row name and engine) never reject the test recipes, which carry Stub filler rows.
- Card 4, the way-forward suffix in `ValidateLocal`: reachable. `Layers` runs it on the local layer.
- Card 5, the retired-key refusal in `PrepareRunSettings`: reachable. `moveRunSettings` holds no move for these keys, and they never enter `retiredLoomKeys`, so no Migrate strips them first. The later `checkLocalDocument`/`ValidateLocal` refusal is the same text.
- Card 5, local-file refusal at load: reachable through `Layers`.
- Card 6, the `LoadTapestryConfig` refusals: reachable.
- **Card 6, plan-perch hard cap: the guarded site recurs unguarded.** `perchHardCap` still reads `cfg.Caps(perch)` from a `LoadConfig` that no longer reads `tapestry.yaml` (finding 2).
- **Card 8, `New`'s non-loom refusal: reachable, though the card calls it unreachable.** Darn reaches it on every darn build between cards 8 and 11 (finding 1).
- Card 9, the batcher check in Preflight: reachable. `reconcileHub` is warn-only (preflight.go:112), so a failing seam is reached even over a non-repository cwd.
- **Card 9, the batcher check's cancelled-context row: the test does not reach the branch it pins.** `entryErr` returns first (finding 4).
- Card 9, the retired row Tapestry-Preflight → Preflight: reachable through `renameRetiredRows` in `armAt`, on both the `lym loom` and `ArmAt` paths.
- Card 11, Record-Write's Stuck on a missing board entry: reachable. Preflight reads no board (preflightshed holds no board import), so a removed entry reaches the row.
- Card 11, the `.tmp-` sweep, write-missing-only and commit-on-resume: reachable through crash states.
- Card 11, `recordWriteEntry`'s relative-path and nil-seam refusals: reachable in the test.
- Card 13, the post-session guard in `darnWrite.Call`: reachable, by a writer that damages the record.
- Card 13, `resolvePlainGateClosure` refusing a nil `DiscussionHistory`: reachable in the test.
- Card 13, the gate branch for a record engine other than `DiscussionWrite`: reached by the test only. After card 13 no shipped recipe whose record row is `RecordWrite` carries a `discussion` gate. This is generality, not a finding.
- Card 15, the missing-record halt, the restore-from-HEAD halt, the missing-support-log halt and the stale-marker removal: reachable.
- Card 15, the stop marker after a Done handoff or a gate-failed Stuck: reachable. A session error or an empty pointer returns before the marker read; the next spawn removes the marker, as D6 allows.
- Card 16, no record with no status file: reachable only in the crash window between the seed write and the status seed.
- Card 16, no record at Preflight or Record-Write, and on a later row: reachable.
- Card 16, "differs from its committed copy" and "missing a historical entry": reachable.
- **Card 16, "not committed at HEAD": reachable on a path the card says cannot occur, where its way forward fails** (finding 3).
- Card 19, `refuseUnservedRecipe` for a batten seed: reachable through an explicit run-id.
- Card 19, the same refusal for a `beam` seed: reachable. `ReadSeed` and `ParseSeed` never validate the recipe (seed.go:117–145).
- Card 19, `refuseVerbOffRecipe` on darn: reachable. The four plan verbs arm through `resolvePersistentPreRun`.
- Card 19, the refusal in `resolveReviewTarget`: reachable.
- Card 14, the nil-commit refusal in `runAndCommit`: a fail-loud wiring guard (`checkSeams` and `requireSeam` run first), so it is exempt.
- Card 17, the stat-error return in `runRecordBullets`: a raw I/O error, so it is exempt.

**Findings**

1. MEDIUM, design, card 8 (`08-one-loom-builder.md`), the Intent sentence on `New`'s non-loom refusal.
   - Card 8 calls the refusal a fail-loud wiring guard that takes no reaching test, because `newShed` branches pawl off and card 19 refuses other recipes.
   - Darn does not declare `contract: loom` (or `verify_source`) until card 11, so every darn build, `New(shedrun.RecipeDarn, …)`, reaches the refusal between cards 8 and 11. Card 8's own darn tests reach it too.
   - The "card 19 refuses before any wiring" premise also lands only at card 19.
   - Evidence: `contracts/recipes/darn-recipe.yaml:61–66` has no `contract` key; card 3 says "Darn declares the contract in card 11"; card 11 adds it.

2. MEDIUM, design, card 6 (`06-tapestry-config-module.md`), `perchHardCap`.
   - The plan-perch hard cap the circling verbs enforce is read from a config that never loads `tapestry.yaml`.
   - Card 6 gives the plan fields the tag `yaml:"-"`, has `LoadConfig` never read `tapestry.yaml`, and applies `WithTapestry` only in `wire` and `reviewBudget`.
   - `perchHardCap` loads `loomengine.LoadConfig(dirs, "loom")` and returns `cfg.Caps(perch).Hard` (`internal/loomcli/circling.go:201–209`, called from circling.go:187). Card 6 only reroutes its way forward through `TapestryConfigSpec()`.
   - So for Plan-Bouncer the circling hard-cap check compares against the shared fallback, not a tuned `tapestry.plan_hard_max_bounces`.

3. MEDIUM, consistency, card 16 (`16-decision-add-by-rows.md`), the "Not committed at HEAD" refusal.
   - The card says only a crash inside Record-Write leaves this state, and names `lym loom resume` as the way forward.
   - The state also occurs for a darn run in flight that halted at Darn-Write when its old discussion gate was exhausted. `NewDarnDiscussionGate` commits the record only on a clean check (`internal/loomshed/gates.go:213–229`). Darn-Write's own commit covers the landing directory only (`internal/loomcli/darnwiring.go:152–155`). The gate is on the Darn-Write row (`darn-recipe.yaml:101`).
   - That run resumes at Darn-Write, not Record-Write. After cards 13 and 15, Darn-Write commits nothing and runs no committed check. So the named `resume` never commits the record, and the refusal repeats on every call.
   - A way forward that can succeed is `lym loom goto --to Record-Write` then `resume`, since Record-Write commits an existing pair without overwriting it.

4. LOW, design, card 9 (`09-drop-tapestry-preflight.md`), the "cancelled context and a failing seam" row of `TestPreflight_BatcherCheck`.
   - The row never reaches the cancel-wins branch after the batcher check. `preflightProducer.Call` returns at `entryErr` before the reconcile and the seam run (`internal/preflightshed/preflight.go:107–109`).
   - To pin that branch, the seam itself must cancel the context and then fail.

Tool calls:
1. grep for readers of `PerchPlan`, `ResolveReview`, `ResolveJudge`, `PlanTable` and `LoadConfig` outside loomengine, to find plan-value readers card 6 leaves unrouted; the manifest held only the `wire` call sites.
2. Read `perchHardCap` (circling.go), `reviewBudget` (arm.go), `codeBudgetZero` (landingdeps.go) and the webstercli `LoadConfig` caller, to see which ones read plan caps.
3. grep for any board read in preflight or preflightshed, the `perchHardCap` callers, and darn-recipe's discussion and subject lines, to check whether Record-Write's missing-entry Stuck can be reached and where darn's gate sits.
4. Read `preflightProducer.Call`, to confirm where the batcher check and the cancel checks can sit.
