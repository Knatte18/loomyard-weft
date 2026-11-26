```yaml
lens: code-affected-set
outcome: reported
```

Task: trace what the diff's changed keys, messages, signatures and behaviours force elsewhere, and report what is left wrong.

**Affected sets**

| Change | Set found | Covered? |
|---|---|---|
| Flat loom and tapestry keys moved to nested paths | Non-test, non-testdata search for flat names across the repo | Covered. The only hit outside the move tables is `tools/tapestryrename/tables.go:88`, a historical rewrite table, which I did not treat as a finding. |
| Keys named in full (D3, `PATTERN-config-key-names`) | Go comments and help text, `docs/overview.md`, contracts, pattern | Not covered: finding 2 |
| Declared PATTERN changes | `PATTERN.md` lines for `PATTERN-config-layers` (changes) and `PATTERN-config-key-names` (adds), plus the config-layers background | Both landed with the declared text |
| Other pattern background rule-line edits | Key renames in the pattern diff | I saw no contradiction with the overview lines I know. I did not compare each line against `PATTERN.md`. |
| Every loading verb now reconciles, not start verbs only | `docs/overview.md`, `docs/shared-libs/configengine.md`, `configcli` help and `doc.go`, `preflightshed/doc.go`, `hubreconcile` docs | `hubreconcile` covered; the rest are not: finding 1 |
| Every reconcile now moves or drops old keys | `pattern/PATTERN-config-layers.md`, `configsync` and `hubreconcile` docs, `configcli` help and `doc.go` | `configcli` not covered: finding 1 |
| `reconcileWayForward` text | Its integration test; searched docs for the claim | Covered |
| `Spec.MovedKeys` refusal | `docs/shared-libs/configengine.md` "Moved keys", `configengine/doc.go` | Covered |
| `CheckLocal` signature, nested budget key builders | Compile-checked callers, tests | Covered by the gate, not re-traced |
| Stencil wording | The contracts diff pieces are outside the manifest | **Not checked** by this lens |
| Commit messages and departures | No `Pattern-Departure:` trailer in the commit log | Nothing to judge |

**Findings**

1. **MEDIUM, consistency.** Docs and help text still describe the old reconcile behaviour.
   - **Start verbs only.** These say a start verb is what runs the walk after a binary change. Now every verb that loads the loom or tapestry config runs it.
     - `docs/overview.md:360`: "the hub config walk a start verb runs once after a binary change".
     - `docs/shared-libs/configengine.md:148`: "the hub reconcile the first start verb after a binary change runs".
     - `internal/preflightshed/doc.go:14`: "the next start verb or Preflight row retries the reconcile" (minor).
   - **Old keys kept.** `internal/configcli/configcli.go` (about lines 726-729, `lym config reconcile` help) says this verb and `--unset` are the only ways to remove a key the template lacks, and that other reconciles keep such a key. `internal/configcli/doc.go:70` says the same. Every reconcile now moves a move-table key, or drops it when it is null or its new path is set. That verb is also D4's way forward, and its help does not mention the move.
   - Evidence: the lines above, against `internal/hubreconcile/doc.go` and `pattern/PATTERN-config-layers.md` as the diff leaves them.

2. **BLOCKING, design** (one finding about the naming method). Config keys are still named without their module, or bare, against `PATTERN-config-key-names` ("in a message, comment, doc or stencil: in full") and D3.
   - **Help text added by the diff:** `internal/loomcli/validate.go`, "the most parts parts.max allows".
   - **Go comments added by the diff:**
     - `internal/loomcli/doc.go:59` (`discussion.review_ask_parent`, `code.review_ask_parent`) and its `review.amend_max_bounces` line.
     - `internal/loomcli/landingdeps.go:459` and `:498` ("loom.yaml's pr.review_ask_parent").
     - `internal/loomcli/circling.go` ("tapestry.yaml's plan.hard_max_bounces").
     - `internal/loomcli/run.go` (`friction.timeout_min`) and `internal/loomcli/wiring.go` (`friction.model`, `plan.timeout_min`).
     - `internal/hardencli/cli.go` ("loom.yaml's friction.model").
     - `internal/loomengine/config.go` (`discussion.advisors`, `code.reviewer_fan`) and `internal/loomengine/caps.go` (`review.max_bounces`).
     - `internal/loomengine/review.go:57-126` (`review.assist_reviewer_timeout_min`, bare `review_parent_timeout_min`).
   - **Docs lines added by the diff:** `docs/overview.md:1693` (bare `fixer_fan`) and `:1695` (bare `assist_reviewer_fan`).
   - **Existing package docs left bare:** `internal/driftengine/doc.go:16,90,96` (`judge_ask_parent`, `judge_interactive`) and `internal/hardenshed/doc.go:10` (`auto_accept`, `reviewer_fan`, `fixer_fan`, `tree_scope`).
   - This list comes from a regex search over selected leaf names, so it is a sample rather than the full set.

**Tool calls**
1. `git grep` for flat key names: the manifest held no repo-wide search. It failed on a bad pathspec and returned nothing.
2. Read `159-pattern.diff`: the pattern diff piece was not in the manifest.
3. `git grep` for remaining flat names and key builders: the corrected rerun of call 1.
4. `git diff` added lines and `git grep` for keys named without a module: the manifest did not cover prose or comments across packages.
5. Search for start-verb reconcile claims, the `PATTERN.md` diff and `docs/overview.md` around line 1690: none were in the manifest.
6. `configcli` reconcile help, the shared-libs configengine doc and package docs: none were in the manifest.
