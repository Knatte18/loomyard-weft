```yaml
lens: repo-invariants
outcome: reported
```

Task: the repo-invariants lens over cards 1–17. I checked reflective seam tests, tier tags, doc text the cards make false, and whether each card can pass its own gate.

**Per card, each invariant checked:**
- **Card 1:** seam fields held (no reflective test covers `Status`); tier tags held; doc held (`shedengine/doc.go` is in Uses, and the state-constant comment drops "six"); gate order held.
- **Card 2:** tier0 held; `shedrun/doc.go` is extended, held; reachable from production through cards 7 and 14, held.
- **Card 3:** tier0 held for the new test file; `PATTERN-leaf-packages.md` section is extended, held. PATTERN-test-only-code is **broken**: `runfsm.Order` has no production caller in any card.
- **Card 4:** tier1 held; `fabricengine/doc.go` is in Uses, held; both functions have production callers through cards 5, 7 and 14, held.
- **Card 5:** gate order **broken**: the planned `battenrecipe` import closes a test import cycle. `pairteardown/doc.go` is extended, held.
- **Card 6:** doc text **broken**: board-status docs still say "running · Worktree-Create" and "the shed state", and the card lists none of them.
- **Card 7:** `docs/overview.md` cancel paragraph and `RunLockedError` text are listed, held. `cancelFake` covers the new seams, held; no reflective test over `cancelDeps`.
- **Card 8:** every test that names `cancelMarker`, `shedrun.CancelMarker` or `Marker:` is listed, held. The remaining production users (`teardown.go:196`, `dobbyreads.go:187`) are listed, held. Doc comments are listed, held.
- **Card 9:** held. It relies on `ReadBattenRecord` and the cleanup texts it lists.
- **Card 10:** held. `battenCLI.runVerdict` is nil-legal and no reflective test walks `battenCLI`.
- **Card 11:** test seam **broken**: the `dobbyengine` import allowlist test fails once `runfsm` is imported. `dobbyengine/doc.go` is listed, held.
- **Card 12:** held. `loomCLI.runVerdict` is nil-legal; `loomcli/doc.go` is in Uses.
- **Card 13:** held. A pure move with production callers.
- **Card 14:** held. `tenderengine` has no import allowlist; both doc.go files are listed; the new fields are nil-legal.
- **Card 15:** reflective `TestLandingDeps_EveryFieldPopulated` and the envkit nil-seam tests are named in Uses and the field is filled, held. tier1 held.
- **Card 16:** the same reflective tests are named and `RecordsDir` is filled, held.
- **Card 17:** `PATTERN.md`, the background file and the `lymdirs` doc are listed, held.

**Findings:**

1. **BLOCKING · design · card 5.** `ReadRun` in `pairteardown` uses `battenrecipe.RowsBeforeRunShed()`, `battenrecipe.NameLand` and `battenrecipe.NameWorktreeTeardown`. Importing `battenrecipe` there closes a test import cycle, so the card cannot pass its gate.
   - The cycle: `battenrecipe` → `preflightshed`, whose in-package tests import `hubforge`, which reaches `pairteardown`.
   - Evidence: `internal/pairteardown/followup.go:18-25` already records this cycle and keeps `landRow` and `worktreeTeardownRow` as local constants, pinned by `TestBattenFollowUp`. `go list` showed that `preflightshed` has in-package test imports of `hubforge`, and `hubforge` reaches `internal/pairteardown`.
   - Fix: take the row names from `pairteardown`'s own pinned constants, or have the callers pass the row facts in. The overview's runfsm-imports decision already says callers fill the row facts.

2. **BLOCKING · consistency · card 11.** `RunView` gains `runfsm.State` and `runfsm.Verdict`, so `dobbyengine` imports `internal/runfsm`. `TestToldGeometryInvariant_AllowlistOnly` fails, because `internal/runfsm` is not in `dobbyengineAllowedImports`, and card 11 does not list that test.
   - Evidence: `internal/dobbyengine/seam_enforcement_test.go:20-36` and `:46`.
   - Fix: add `internal/dobbyengine#dobbyengineAllowedImports` to card 11's Edit list, with `internal/runfsm` added.

3. **MEDIUM · consistency · card 6.** Card 6 changes the state word the board status carries, from the shed state to the FSM state (`starting`, `halted`, `landing`). These docs still state the old words and no card lists them:
   - `internal/battencli/cli.go:119` says the claim writes "running · Worktree-Create". Card 8 edits `cli.go`, but only for the marker comments.
   - `internal/boardengine/runlock.go:13` and `:16` say `RunStatus` composes "the shed state and the producer".
   - `contracts/orchrole/orch-role.md:312` names `running · Worktree-Create` as a pre-loom status.
   - `docs/overview.md:1338-1339` gives `awaiting · Code-Bouncer` as the example. Card 7 edits `docs/overview.md`, but only its batten cancel paragraph.

   Fix: add these to card 6's targets with the FSM wording.

4. **MEDIUM · design · card 3.** `plan:internal/runfsm#Order` (`func Order() []State`) is reached only by `TestDerive`; no card's production code calls it. `cmd/lym/testonlycode_linux_test.go` (PATTERN-test-only-code) fails a production function that no main package reaches.
   - Evidence: `_lym/plan/03-runfsm.md:16` and `:57`; no card's Uses names `runfsm#Order`; the guard is at `cmd/lym/testonlycode_linux_test.go:1-4`.
   - Fix: have `Derive` iterate `Order()` and say so in the Intent, or drop `Order`.

Tool calls:
1. Grepped for the test-only-code guard and for reflective seam tests: the manifest had neither.
2. Read the guard's header, and grepped docs and patterns for cancel-marker and records-removal text: needed to see which doc text the cards make false.
3. Read `battencli/cli.go:112-124` and grepped for board-status state-word docs: card 6's affected docs were not in the manifest.
4. Grepped for the remaining `CancelMarker` and `cancelMarker` users, the board state-word docs, and the plan's callers of `runfsm.Order`: needed to check card 8's completeness and card 3's reachability.
5. Ran awk to name the test functions enclosing each cancel-marker reference, and read `docs/overview.md:1335-1341`: needed to confirm card 8 lists every test it breaks, and the overview's example.
