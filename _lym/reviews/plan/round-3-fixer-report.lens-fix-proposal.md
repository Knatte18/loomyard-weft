```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each of the reviewer's proposals the fixer carried out.

**Findings**

1. **MEDIUM. Card 02 Intent, rewritten sentence:** "Every row of `TestArmFromSeed_RefusalEnvelopeKinds` that seeds an unknown recipe … asserts that text as its suffix and counts one way-forward clause. The kind-less branch … also asserts `wantSuffix` …"
   - **Summary:** The proposal adds only a suffix assertion to the kind-less branch, so the "unknown recipe on status" row still never counts its way-forward clauses. The rewritten sentence now claims it does.
   - **Evidence:** In `internal/shedcli/cli_test.go`, the `wantKind == ""` branch (lines 274–278) returns before the `strings.Count(msg, "way forward:")` check at line 293. The "unknown recipe on status" row sets no `wantWayForwardCount` and no `wantKind`. The proposal names only `wantSuffix` for that branch. As written, either the branch must also check `wantWayForwardCount` when the row sets it, and the status row must set it to 1, or the sentence must stop saying the status row counts one clause.

**Proposals checked with no case against them**

- **F1, card 06:** After card 6, `LoadConfig` of a migrated `plan_lenses` only warns. `warnUnknownKeys` in `internal/configengine/config.go` logs `logger.Warn` and returns nil. So dropping the `PlanLenses` conditions is right, and the migration rows still pin the rewrite.
- **F1, cards 09, 10 and 11:**
  - Card 9: removing Tapestry-Preflight takes Friction-Reflect from 16 to 15, Plan-Write to 5 and Plan-Burler-Review to 6.6.
  - Card 10, applied after card 9: Drift-Verify takes 10 and the Code group 11, which matches "the number Code-Bouncer held".
  - Card 11: darn becomes Record-Write 2, Drift-Baseline 3, Code-Bouncer 7 and Friction-Reflect 11.
  - The arithmetic of all three holds.
- **F2, card 06:** The hard-below-soft refusal at `internal/reviewbudget/reviewbudget.go:150` prints `%s config key %q` with `caps.HardKey`. With `plan_hard_max_bounces: 0` that key is `review_hard_max_bounces`, routed through `RunWideSpec`, so the prefix `loom config key "review_hard_max_bounces"` follows. The proposed row is sound.

**Tool calls beyond my context**

- `internal/configengine/config.go:296-320`: whether `warnUnknownKeys` refuses or warns. My context showed only the start of the function.
- `CheckResolved` refusal format strings in `internal/reviewbudget`: the F2 prefix claim. My context had not read the code.
- `internal/shedcli/cli_test.go:166-240` row fields: the kinds and suffixes of the F3 rows. My context showed only the harness branch.
