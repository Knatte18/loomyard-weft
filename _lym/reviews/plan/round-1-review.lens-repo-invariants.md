```yaml
lens: repo-invariants
outcome: reported
```

Task: review cards 1–13 through the repo-invariants lens (test seams and tier tags, docs a card makes false, cards that cannot pass their own gate).

**Per card, each invariant: held or broken**

- **Card 1**
  - Seam: none touched. Held.
  - Tier: `TestState_Valid` gets a new row. Held.
  - Doc: broken. The comment on the const block says "The six legal State values" (`internal/shedengine/status.go:13`). The card targets only `State.valid`; the const block sits under Uses as `StateDone`.
  - Order: held.
- **Card 2**
  - Seam: none touched. Held.
  - Tier: `state_test.go` and `act_test.go` are both tier0. Held.
  - Doc: creates its own `doc.go`. Held.
  - Order: imports only the standard library. Held.
- **Card 3**
  - Seam: none touched. Held.
  - Tier: `read_test.go` is tier1, and the new battencli test is tier0 with its file not pinned. Held.
  - Doc: held.
  - Order: held. runfsm imports `hubgeom`, which depends on `dobbyengine`. No package that `hubgeom` depends on imports runfsm yet.
- **Card 4**
  - Seam: none touched. Held.
  - Tier: `TestResumeVerb` gets new rows. Held.
  - Doc: `resumeCmd`'s Long text stays true. Held.
  - Order: held.
- **Card 5**
  - Order: **broken**, an import cycle.
  - Doc: broken. `dobbyengine/doc.go` is listed only under Uses.
  - Seam: no reflective test marks `RunView`. Held.
- **Card 6**
  - Doc: broken. `cancelCmd`'s Long text, the comment on `registeredBattenAlive`, and the cancel-marker lines of `dobbyengine/doc.go` all become false.
  - Tier: held.
  - Order: held, since `hubgeom` does not depend on `pairteardown`.
- **Card 7**
  - Doc: broken. `cancelCmd`'s Long text and envelope description say the cancel removes the records.
  - Order: held.
- **Card 8**
  - Order: **broken**. `clearDoneRun` shares `judgeStatus`, so card 7's new `TestClearDoneRun` row fails at card 8.
  - Doc: broken. `cleanupCmd`'s Long text becomes false.
- **Card 9**
  - Doc: held. The `doc.go` sits under Extends, and the comment on `RemovePair` is an Edit target.
  - Tier: tier1. Held.
  - Order: held.
- **Card 10**
  - Seam: no reflective test marks `tenderengine#Deps`. Held.
  - Doc: Extends on both `doc.go` files, and `pauseCmd`/`resumeCmd` are Edit targets. Held.
  - Order: held.
- **Card 11**
  - Seam: **broken**. The new `Publish` func field is left nil in `envkit.LandingDeps` and in batten's wiring.
  - Doc: broken. `pairteardown/doc.go` and the comment on `RemovePair` describe the record as sitting in scratch.
- **Card 12**
  - Seam: **broken**. `envkit.LandingDeps` gets no `AttemptsDir`, and the constructors now refuse an empty one.
  - Doc: held, with Extends on `landingshed/doc.go`.
- **Card 13**
  - Doc: held. It edits the PATTERN entry, its background file and `DotLymDirName`'s comment.

**Findings**

1. **BLOCKING · design · card 5 (`05-dobby-pass-asks-act-rule.md`, `dobbyengine#RunView`) — card 5 cannot compile.**
   - Card 5 adds `Records runfsm.Records` to `RunView`, so `dobbyengine` imports runfsm.
   - Card 3's reader puts `hubgeom` into runfsm's imports, and `hubgeom` imports `dobbyengine`. That closes the cycle `dobbyengine` → runfsm → `hubgeom` → `dobbyengine`.
   - Evidence:
     - `go list -f '{{.Imports}}' ./internal/hubgeom` lists `internal/dobbyengine`.
     - `internal/hubgeom/dobbygeom.go:29` returns `dobbyengine.Geometry`.
     - `03-runfsm-reader-batten-run.md`, Intent paragraph on `ReadOwner` and `00-overview.md` Shared Decisions: runfsm imports `hubgeom`.
   - Fix: put the reader in a package `dobbyengine` does not import (for example a subpackage), keeping `Records` in the stdlib-only core. Or have the reader take `parentOf` as an argument instead of importing `hubgeom`.

2. **BLOCKING · consistency · card 8 (`08-cleanup-cancelled-record.md`, `battencli#judgeStatus`) — card 8 breaks card 7's test and D3.**
   - `judgeStatus` is also the done check of `clearDoneRun` (`internal/battencli/cleanup.go:222`).
   - Card 8 makes `judgeStatus` take a finished cancel's record as a candidate. Then `ClearDoneRun` (the dobby's done-run clear) removes a cancelled record and discards its task branch on origin (`cleanup.go:139-150`, `237-241`).
   - That contradicts D3's "the dobby's done-run clear never takes a cancelled run".
   - It also fails the `TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits` row card 7 adds, which asserts the cancelled run is left (`07-cancel-finish-keeps-record.md`, Tests sentence).
   - Card 8 lists neither `clearDoneRun` nor that test.

3. **MEDIUM · design · card 11 (`11-pull-request-record-tracked.md`, `landingshed#PullRequestDeps`) — the new `Publish` seam breaks the seam tests.**
   - The new func field must be filled in `internal/testkit/envkit#LandingDeps`. Its `PullRequest:` block is at `envkit.go:224`, and `NilSeams` reports nil func fields (`envkit.go:773-776`).
   - Batten's wiring has no landing, so `battencli`'s `intentionallyNil` map needs a `"Landing.PullRequest.Publish"` entry (`internal/battencli/wire_seams_test.go:38`, beside the entry at line 111).
   - The card lists `TestFullEnv_FillsEverySeam` and `TestWire_EverySeamFilled` under Uses only. It edits neither the envkit builder nor the map, so its own gate fails.

4. **MEDIUM · design · card 12 (`12-fixer-attempts-tracked.md`, `landingshed#Deps`, `NewFinalize`, `NewParentAlign`) — the empty-`AttemptsDir` refusal breaks every envkit landing test.**
   - The card makes both constructors refuse an empty `AttemptsDir`.
   - The shared fixture `envkit#LandingDeps` (`internal/testkit/envkit/envkit.go:182-200`) sets `ScratchDir` but not `AttemptsDir`, and no card lists it. Every test that builds a landing through envkit would hit the refusal.

5. **MEDIUM · consistency · card 5 (`dobbyengine/doc.go`, Uses only) — the dobby's doc describes the pass rules card 5 replaces.**
   - `internal/dobbyengine/doc.go:154-175` and `286` describe: no pass while the cancel marker exists, a paused run driven once its work reached its parent, and continue passes while a pause covers the run.
   - Card 5 replaces all three with the act rule, and a covered run gets no continue pass. The doc lists none of it, and the card names `doc.go` only under Uses.

6. **MEDIUM · consistency · cards 6–7 (`battencli#battenCLI.cancelCmd`, not listed) — the cancel command's help text becomes false.**
   - The Long text says the cancel "removes the run's records and scratch and commits that removal" and "leaves its marker" offline (`internal/battencli/cancelcmd.go:54-68`). The envelope keys `records_removed` and `push_spawned` are described the same way.
   - Card 6 replaces the marker with the intent, and card 7 keeps the record, writes its finish and pushes it synchronously.
   - Neither card lists `cancelCmd`.

7. **LOW · consistency · card 8 (`battencli#battenCLI.cleanupCmd`, not listed) — the cleanup command's help text becomes false.**
   - The Long text says cleanup removes "every done run" and skips a run that is "running, paused, blocked, awaiting or failed" (`internal/battencli/cleanup.go:628-637`).
   - Card 8 adds removing a finished cancel's record and refusing it while its pair stands, and does not list `cleanupCmd`.

8. **LOW · consistency · card 11 (`internal/pairteardown/doc.go`, not listed) — pairteardown's text still puts the record in scratch.**
   - Both `pairteardown/doc.go:101-102` and the comment on `RemovePair` (`internal/pairteardown/teardown.go:407-408`) say "the record under the run's scratch directory is still there".
   - Card 11 moves the record to the pair's tracked `_lym` directory and says pairteardown "needs no change".

9. **LOW · consistency · card 6 (`pairteardown#Teardown.registeredBattenAlive`, not listed) — a doc comment still names the marker.**
   - Its comment says "running with no cancel marker" (`internal/pairteardown/teardown.go:713`).
   - Card 6 makes the `cancelMarked` closure in `New` read the intent instead, and does not touch this comment.

10. **NIT · consistency · card 1 (`internal/shedengine/status.go:13`) — the const block's comment stays at six.**
    - The comment "The six legal State values" sits on the const block, which is not a card target.
    - Card 1's Intent rewrites the state-block comment to count seven, but lists only `State.valid` under Edit.

Tool calls:
- grep of the `//lym:seam` markers for the structs the cards change: the manifest did not cover reflective seam tests.
- `envkit.go` around `LandingDeps` and `NilSeams`: the shared fixture builder was not in context.
- grep of envkit's `PullRequest` fill and the `intentionallyNil` entries: needed to confirm what the seam tests check.
- `wire_seams_test.go` map names, plus the doc text in `dobbyengine/doc.go` and the battencli, tendercli and PATTERN mentions: doc text was outside the manifest.
- grep of the import cycle: none made; the `hubgeom` import list was already in context.
