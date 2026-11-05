```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked three invariants: the reflective seam tests and test-file tiers, the doc text each card's Intent falsifies, and whether each card can pass its own gate after the cards before it.

**Per card**
- **Card 1** fabric-trailer-commits: seams held (adds no seam struct field). Tier held (tier1, new test). Doc held (fabricengine doc.go Extends). Order held.
- **Card 2** fabric-checkout-states: seams, tier, doc and order all held.
- **Card 3** drift-capture-diff: seams held. Tier held (tier0). Doc held (driftengine doc.go). Order held.
- **Card 4** drift-config-keys: seams held (configengine#Spec carries no `//lym:seam`). Tier held. Doc held (all three doc.go files Edit). Order held.
- **Cards 5 and 6** (the parsers): all held. Card 6 Edits `PATTERN.md` and the sole-parsers background file.
- **Card 7** drift-records-guard: seams held (Deps and Paths, every `//lym:seam` test listed in Uses). Tier held (`records_test.go` is tier0). Doc held. Order held.
- **Card 8** drift-seal-baseline-checks: seams held. Tier held. Doc held. Order held.
- **Card 9** drift-current-capture: seams held. Tier held (`capture_test.go` is all tier1). Doc **broken**: `pattern/PATTERN-daemon-wakeups.md` is not updated (F1). Order held.
- **Card 10** drift-containment-check: seams held (shedrecipe#Env is not extended). Tier held. Doc held: recipe comments and shedrecipe doc.go both change. Order held.
- **Card 11** webster-runlevel-entry-report: seams held (RunDeps unchanged). Tier held. Doc held. Order held.
- **Card 12** drift-expect-producer: seams, tier, doc and order all held.
- **Card 13** drift-expect-rows: seams held. Doc **broken**: `internal/loomrecipe/doc.go` (F2) and `docs/overview.md` (F3). Order held.
- **Card 14** drift-check-verb: tier held (tier1). Doc held (driftcli doc.go). Order held.
- **Card 15** drift-judge-seat: seams held. Doc held. Order held.
- **Card 16** drift-verify-lines: seams held (EndRoles; seam tests in Uses). Doc **broken**: `internal/loomrecipe/doc.go` (F2) and `docs/overview.md` (F3). Order held.
- **Card 17** drift-verify-command: all held.
- **Card 18** drift-baseline-await: doc **broken**: `docs/overview.md` (F3). Everything else held.
- **Card 19** drift-amend-defer: tier held (`TestAmend` tier0 joins tier0 `verbs_test.go`; the tier1 test is in the external package). Doc held. Order held.
- **Card 20** drift-verdict: all held.
- **Card 21** drift-writer-stencils: all held.
- **Card 22** driftcli-retire-verbs: doc **broken**: `docs/overview.md` (F3). Everything else held.
- **Card 23** drift-retire-recheck-engine: all held.
- **Card 24** drift-retire-old-seats: seams held. Doc held (`PATTERN-parent-directive.md` is Edited). Order held.
- **Card 25** driftengine-retire-acceptance: seams held. Order **broken**: two duplicate deletes (F4).
- **Card 26** drift-retire-skip-keys: all held.
- **Card 27** drift-retire-verify-gate-key: held, apart from one gap. It covers the gateslot doc.go, gatecli doc.go (the `drift-verify:` example at `internal/gatecli/doc.go:332`), loomshed doc.go, `PATTERN-gate-slot.md`, `PATTERN-test-speed.md` and `docs/dotnet-hub.md`. Before card 16 lands, PATTERN-gate-slot.md:58 and :62 describe the Drift-Verify gate site; only card 27 fixes that text, and card 27 comes after 16. That gap is transient, so it is not raised as a finding.

**Findings**

- **F1 — MEDIUM, consistency.** Location: card 09-drift-current-capture.
  - Summary: `CaptureCurrent` adds a new unbounded lock wait ("waits for the capture lock with no bound of its own, sleeping and retrying"). `pattern/PATTERN-daemon-wakeups.md` lists every unbounded wait under "What counts", and no card adds this one or Edits the file. The card also never states the one-second floor and the backoff for this wait.
  - Evidence: `pattern/PATTERN-daemon-wakeups.md:15` lists only the `internal/driftengine/wait.go` capture wait; card 09 lines 47–48.

- **F2 — MEDIUM, consistency.** Location: cards 13-drift-expect-rows and 16-drift-verify-lines.
  - Summary: `internal/loomrecipe/doc.go` states each recipe's row order without Drift-Expect, and says "Drift-Recheck's on_stuck names Drift-Verify". Card 13 inserts Drift-Expect and card 16 drops that `on_stuck`. Card 13 lists the file only under Uses, and no card Edits it.
  - Evidence: `internal/loomrecipe/doc.go:15`, `:17`, `:19`.

- **F3 — MEDIUM, consistency.** Location: cards 13, 16, 18 and 22.
  - Summary: no card Edits `docs/overview.md`, which these cards falsify:
    - Line 279 lists the `lym drift` verbs `scope`, `accept`, `withdraw`, `compare` and `waive` (card 22 removes them) and omits `check` and `amend`.
    - Line 616 says Drift-Recheck compares against the accepted profile (removed by cards 16 and 25).
    - Lines 614–615 and 617 give the drift row placement without Drift-Expect (card 13).
    - Line 618 says Drift-Baseline spawns an agent to decide the scope (card 18 removes the agent).
  - Evidence: `docs/overview.md:279`, `:614`–`:619`.

- **F4 — BLOCKING, design.** Location: card 25-driftengine-retire-acceptance.
  - Summary: card 25 deletes `internal/driftengine#verifyFixture.agentAccept` and `#verifyFixture.sessionAccept`, which card 16 already lists under its own `**Delete:**`. Card 25 cannot run from the state card 16 leaves, and `delete-target-gone` fires at Webster's dispatch.
  - Fix: drop both refs from card 25's Delete list.
  - Evidence: card 16 lines 74–75; card 25 lines 48–49; `internal/driftengine/verify_test.go:224` and `:234`.

**Tool calls**
1. Grepped `pattern/`, `PATTERN.md` and `docs/*.md` for drift terms: to find doc text the cards falsify, which the manifest did not cover.
2. Counted `tiergate.Require` per drift test file: to check each file's one-tier invariant.
3. Read the tier lines of the drift test files and the `//lym:seam` markers: to check the tiers of the files the new tests join.
4. Read the `wiring_seams_test.go` marker block and grepped `//lym:seam` across the repo: to find the reflective seam tests over the structs the cards change.
5. Grepped `doc.go` files for drift row and verb text: to find package docs no card lists.
6. Grepped the `shedadapters` and `gatecli` doc.go files: to confirm the two remaining doc hits.
