---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/09-reconcile-failure-warns.md:56
    summary: The reworked loom resume row asserts the output lacks re-run this command, but card 9's own EnsureOrWarn Warn prints the WorktreeError text ending in that phrase to stderr, so the row fails against a correct implementation; the Intent also claims the loom load refuses with its own way forward, which it does not.
    fix: In card 9's Intent, rewrite the TestLoomResumeReconcilesTheHubConfigBeforeArming sentence so the broken-loom.yaml step expects exit 1, output naming the broken file, the Warn's reconcileWayForward text and the loom config load's own error for that file, and no stamp written. Drop the assertion that the output holds no re-run this command. The proof that the reconcile no longer stopped the verb is the loom load's error appearing after the Warn. In the Bound paragraph, replace "refuses with its own way forward as before" with a statement that the operator's way forward for a broken file is the Warn printed above the load's refusal in the same output.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/09-reconcile-failure-warns.md:53
    summary: Card 9 puts its logcapture assertions in subtests of TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas and TestEnsure_HeldLockTimesOutWithoutStamping, both t.Parallel parents, so the swapped process logger is shared with concurrently running hubreconcile tests and the logs-no-Warn assertion can flake.
    fix: In card 9, drop the Edit targets internal/hubreconcile_test#TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas and internal/hubreconcile_test#TestEnsure_HeldLockTimesOutWithoutStamping. Add the Create target plan:internal/hubreconcile_test#TestEnsureOrWarn_WarnsWithTheWayForward -> func TestEnsureOrWarn_WarnsWithTheWayForward(t *testing.T). Rewrite the two Intent sentences at lines 53-54 to make it one table-driven top-level test that calls logcapture.Capture and never calls t.Parallel, as TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent does. Its rows are a broken hub, where the Warn names the failing file and reconcileWayForward and no stamp exists, then the fixed file, where there is no Warn and the stamp is written; and a held lock, where the Warn names the lock and no stamp exists. Its comment says the logger is process-global. This combination passes lyx loom validate-plan --overlay together with F4, F5 and F8.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/06-sparse-hub-base-reconcile.md:33
    summary: Card 6 swaps yamlengine.Reconcile for yamlengine.Prune on a present base file and says lenses keeps dropping template-equal per-name entries into Dropped as today, but Prune never drops a per-name entry or reports Dropped; only Reconcile does.
    fix: In card 6's first present-file bullet, name the mechanism for the per-name drop. For a module with PerNameMaps, after Prune, drop each per-name entry whose value decodes equal to the template's, with the same equality yamlengine.Reconcile uses for its dropped report, and report it in Result.Dropped as P.<name>. Either factor that comparison out of Reconcile or add it to the present-file path. Add a row to TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate in which a present lenses base holds a template-equal lens beside a tuned one; it expects the template-equal lens absent and reported in Dropped, the tuned one kept, and no StripBase involved.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/plan/08-walk-strips-base-once.md:11
    summary: Card 8's afterHubWideWrite hook runs inside reconcileHubWide's board write, but reconcileHubWide takes only boardDir, primeAnchor and label, so it must gain the walkHooks parameter; its Edit glyph carries no re-sign arrow, and walkHub, which must pass the hooks, is not a target.
    fix: "In card 8's Edit list, replace the line - `internal/hubreconcile#reconcileHubWide` with - `internal/hubreconcile#reconcileHubWide` -> `func reconcileHubWide(boardDir, primeAnchor, label string, hooks walkHooks) (skipped bool, err error)` and add - `internal/hubreconcile#walkHub` after it. Say in the Intent that walkHub passes its hooks to reconcileHubWide. This overlay passes lyx loom validate-plan --overlay."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/03-claude-orch-webster-batcher-tuned-defaults.md:36
    summary: Card 3 says battencli's overview_test.go stays as it is, but TestLoadOverviewThresholds' row no orch.yaml resolves the template thresholds expects Soft 300000 and Hard 400000 from the template and fails after the change.
    fix: In card 3, add - `internal/battencli#TestLoadOverviewThresholds` to the Edit list after the orchengine test lines. Add a Tests bullet saying its no-orch.yaml row expects Soft 150000 and Hard 200000. Narrow the line-36 sentence to the orchThresholds fixture alone. This overlay passes lyx loom validate-plan --overlay.
  - id: F6
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/11-clone-writes-base-marker.md:24
    summary: Card 11 writes the board marker before CloneAndWire's board commit, with no removal when that commit fails, so a failed clone commit leaves the marker untracked. The next walk then sees the marker present and never commits it, and the walk's restore-on-failure bound from card 8 has no counterpart in the clone.
    fix: In card 11's Intent, add that when the clone's board commit fails, CloneAndWire removes a marker it wrote in this call before returning the error, so the next start verb's walk strips and commits the marker itself. Add an assertion to TestRunCLI_CloneEndToEnd, or state why the commit cannot fail after the marker write.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/08-walk-strips-base-once.md:52
    summary: Card 8's new board way forward names lyx config <module> --layer default without saying from the hub's prime worktree. Card 9 surfaces it from lyx loom resume in a task worktree and from the Preflight row in a pair, where configcli's hubWriteRefusal refuses a board write.
    fix: In card 8's WorktreeError.Error sentence, make the board way forward read run lyx config <module> --layer default from the hub's prime worktree, the form reviewbudget.Route and configcli already use for a board file.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/10-config-cli-sparse-hub-base.md:7
    summary: docs/overview.md's config bullet says lyx config reconcile --layer default does the same as reconciling against the live templates, but after cards 6 and 10 it seeds sparse, never fills a present base and only reports the base strip as pending; no card edits that sentence.
    fix: In card 10, add - `docs/overview.md` to the Edit list after internal/configcli/doc.go. Say in the Intent that the config bullet's --layer default sentence states that it seeds absent board files sparse, prunes but never fills a present one, and lists the one-time base strip as pending, which only a start verb's hub reconcile performs. This overlay passes lyx loom validate-plan --overlay.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/09-reconcile-failure-warns.md:59
    summary: Card 9 counts four stamp-gated Ensure call sites, but CloneAndWire's hubreconcile.Ensure at internal/fabriccli/clone.go:130 is a fifth; it still fails the clone on a reconcile error, and the card neither moves it nor says why it stays.
    fix: In card 9's Intent, next to the sentence keeping battencli's Options.Pair call on Ensure, add that CloneAndWire's stamp-gated walk keeps Ensure and its failure, because a clone creates the hub rather than reading one, and a walk failure there already leaves the clone intact for the next start verb. In the ImpactSummary, name the four call sites as the start verbs and the Preflight row.
---

### [MEDIUM] Loom resume row asserts an absence the card's own Warn contradicts

**Location:** `_lyx/plan/09-reconcile-failure-warns.md:56`

**Issue:** The row runs the built `lyx` binary and checks its combined output (`internal/loomcli/smoke_prebootstrap_integration_test.go:205-219`).
The logger defaults to `Warn` on `os.Stderr` outside `go test` (`internal/logger/logger.go:171-176`, `init` at `:367`).
`EnsureOrWarn` logs the `WorktreeError`, whose text ends `then re-run this command` (`internal/hubreconcile/hubreconcile.go:83`).
So the output contains the phrase the row says is absent.
The row's other premise, that the loom load "refuses with its own way forward", is also false: `loomCLI.wire` returns `loomengine.LoadConfig`'s error bare (`internal/loomcli/wiring.go:515-517`).
Only the Warn printed above that error carries the way forward.

### [MEDIUM] Logger capture inside parallel parent tests

**Location:** `_lyx/plan/09-reconcile-failure-warns.md:53-54`

**Issue:** `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas` (`internal/hubreconcile/ensure_integration_test.go:542`) and `TestEnsure_HeldLockTimesOutWithoutStamping` (`:679`) both call `t.Parallel()`.
A subtest that skips `t.Parallel` still runs while the package's other parallel tests run.
`logcapture.Capture` redirects the process-wide logger (`internal/testkit/logcapture`).
A concurrent test's Warn can therefore land in the buffer and break the "logs no Warn" assertion, and a concurrent capture's cleanup can cut this one off.
The package's existing capturing tests are top-level and non-parallel for this reason (`ensure_integration_test.go:802-804`, `:829-830`), and so is `TestLoadOverviewThresholds` (`internal/battencli/overview_test.go:563`).

### [MEDIUM] Prune cannot carry lenses' template-equal drop

**Location:** `_lyx/plan/06-sparse-hub-base-reconcile.md:33`

**Issue:** `yamlengine.Prune` removes unknown units and keeps per-name names, returning `removed` and `kept` only (`internal/yamlengine/layers.go:330-336`).
The per-name drop of template-equal names and its `dropped` report belong to `yamlengine.Reconcile` alone (`internal/yamlengine/reconcile.go:31-33`).
The card keeps the behavior "as today" but names a function that cannot produce it.
`stripBase` does not cover the gap either, because it runs only under `StripBase` on an unmigrated board, while D2 says `lenses` drops on every reconcile.

### [MEDIUM] reconcileHubWide changes signature without the re-sign arrow

**Location:** `_lyx/plan/08-walk-strips-base-once.md:11`

**Issue:** `reconcileHubWide(boardDir, primeAnchor, label string)` (`internal/hubreconcile/walk.go:188`) has no access to `walkHooks`, and `walkHub` calls it without them (`walk.go:122`).
The card's `afterHubWideWrite` seam runs "inside the board write after `ReconcileHubWideAt` returns" (line 46), which is inside `reconcileHubWide`'s `Write` closure.
The hooks have to reach it as a parameter, so the member's signature changes and its glyph needs the re-sign arrow.

### [LOW] TestLoadOverviewThresholds reads the old orch template thresholds

**Location:** `_lyx/plan/03-claude-orch-webster-batcher-tuned-defaults.md:36`

**Issue:** `internal/battencli/overview_test.go:572`'s row "no orch.yaml resolves the template thresholds" expects `{Soft: 300000, Hard: 400000}`.
The card's grep instruction would find it, but the card names this file as one that stays unchanged.

### [LOW] A failed clone commit leaves the marker untracked

**Location:** `_lyx/plan/11-clone-writes-base-marker.md:24`

**Issue:** `CloneAndWire` returns on a failed `b.Commit` and leaves the clone intact (`internal/fabriccli/clone.go:63-66`).
A marker written before that commit stays on disk untracked.
Card 8's walk adds the marker to its commit only when the pass created it, so the marker is never committed.
Every later walk also skips the strip and the stamp staleness that an absent marker would trigger.

### [LOW] Board way forward omits the prime

**Location:** `_lyx/plan/08-walk-strips-base-once.md:52`

**Issue:** A board write from a task session or a pair is refused by `configcli`'s hub write guard (`internal/configcli/hubwide.go:46-58`).
Card 9 surfaces the board error from `lyx loom resume` in a task worktree and from the Preflight row in a pair.
A way forward that omits the prime therefore sends the operator to a refused command first.

### [LOW] docs/overview.md still describes a full default-layer reconcile

**Location:** `_lyx/plan/10-config-cli-sparse-hub-base.md:7`

**Issue:** `docs/overview.md:355-356` says `lyx config reconcile --layer default` "does the same" as reconciling every file against its live template.
After cards 6 and 10 that sentence is wrong, and no card targets `docs/overview.md` for it.

### [LOW] CloneAndWire's Ensure call is left unaddressed

**Location:** `_lyx/plan/09-reconcile-failure-warns.md:59`

**Issue:** `internal/fabriccli/clone.go:130` is a stamp-gated `hubreconcile.Ensure` call that card 9 neither moves to `EnsureOrWarn` nor names as kept.
The card's count of "the four stamp-gated call sites" is therefore wrong by one.

## Rejected

- Way-forward lens, "the Warn may be silent at default verbosity": the logger's stderr threshold starts at `Warn` (`internal/logger/logger.go:367`) and `DefaultOutput` is `os.Stderr` outside `go test`, so `EnsureOrWarn`'s Warn reaches the operator.
- Guard-reach lens, "the marker-withheld branch is unreachable in production": D3 requires the precondition, and the card 6 row reaches it through `ReconcileHubWideAt`'s own options (`StripBase` and `Apply` without `SeedDefaults`), the function's public surface, so it is a named input and test.
- Handler, `orchengine`'s threshold constants beyond D1's template lines: licensed. The const block at `internal/orchengine/config.go:18-21` defines them as the template defaults that the accessors floor back to, so leaving them at the old numbers would make a floored value differ from the template.
- Handler, the Shared Decisions reading "left as it was" as per failing step: D3 already decides that a later worktree failure keeps the committed board and marker. The addition's stated rationale, never a half-written config, holds under that reading, and the plan records it.
- Handler, the stamp-gated `Ensure` at the start verbs and the Preflight row standing in for "when lyx reads a hub's config": D3's rationale names the next start verb's walk as the trigger, and the addition's "a matching stamp costs one comparison" is that fast path.
- Handler, card 10's `base_strip` envelope field: it is the way forward for D3's pending strip report, not new scope.
- Handler, card 8's `--layer default` way forward and its `Board` field: these follow from the new marker error (empty `Module`) and the board-file failure row that D3's failure path needs.
- Card order: no card uses a symbol a later card creates. Card 4 provides `SparseBaseHeader`, `BaseSparseMarker`, `BaseSparseMarkerRel` and `BaseScaffold`; card 5 provides `formerValuesDoc`; card 6 provides `sparseSeed`, `WriteBaseSparseMarker` and the test function that card 7 extends. Cards 8, 10 and 11 use only these.
- Verify coverage: `go build ./...` and `lyx gate test --tags integration ./...` run every targeted package's untagged and integration tests. The hub lists no verify tiers, and the plan spec says the `tmux` tier is compiled, never run, by a plan.
