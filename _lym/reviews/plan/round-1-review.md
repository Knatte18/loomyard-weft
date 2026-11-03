---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/03-runs-strand-beside-status.md (Intent, docs/overview.md bullet); docs/overview.md:424
    summary: Card 3 rewrites only the batten entry of docs/overview.md, leaving the orch entry at line 424 saying that ensuring the runs strand replaces a loom-status strand, which D4 makes false.
    fix: In card 3's Intent, extend the docs/overview.md bullet so it also rewrites the orch entry's clause at docs/overview.md:424 ("ensuring it replaces a `loom-status` strand a pawl run's `lym loom start` put in the overview window first") to say that ensuring it adds the runs strand beside such a strand, as the session's first overview window. docs/overview.md is already a card 3 target, so no target changes.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: internal/reedengine/ownwindow.go:170; internal/orchcli/runs.go:1; internal/orchcli/runs.go:39; internal/reedcli/follow.go:38
    summary: The scope line's sweep of reed and orchcli comments that state one overview per session misses the isOverview doc comment, the runs.go file header, the runsStrandAddSpec doc comment and the lym reed follow help text.
    fix: "Card 1: move `internal/reedengine#isOverview` from Uses to Edit, and add an Intent line saying its doc comment reads that d places a strand in an overview window, one of the session's first windows. Card 3: add Intent lines saying the runs.go file header and runsStrandAddSpec's doc comment call the runs strand's window the session's first overview window (runsStrandAddSpec is already a target, and the header sits in a file card 3 already targets). Card 2: add `internal/reedcli#reedCLI.followCmd` to Edit and `internal/reedcli/doc.go` to Uses, and add an Intent line saying the follow help's relevant window reads: the newest live agent window, else the lowest-index overview window. The combined overlay of these target changes passes lym loom validate-plan --overlay."
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/01-reed-overview-order-key.md (Tests, RelaunchingOrReplacingEitherOverviewKeepsTheOrder)
    summary: The relaunch row says it relaunches pawl with AddStrand IfAbsent and cmd sleep 300, but the IfAbsent relaunch launches the stored ResumeCmd else Cmd, which is exit, so the relaunched pane dies again.
    fix: In card 1's RelaunchingOrReplacingEitherOverviewKeepsTheOrder bullet, replace "relaunch it with `AddStrand` `IfAbsent` and cmd `sleep 300`" with "relaunch it with `AddStrand` `IfAbsent`, which relaunches its stored command `exit`", so the row asserts only the order after the relaunch.
---

### [MEDIUM] Card 3 leaves the orch entry of docs/overview.md stating the replaced D4 behaviour

**Location:** `_lym/plan/03-runs-strand-beside-status.md` Intent; `docs/overview.md:424`

**Issue:** D4 stops `ensureRunsStrand` from replacing a loom status strand.
Card 3 targets `docs/overview.md`, but its Intent rewrites only the batten entry's sentence at line 687.
The orch module entry at line 424 carries the same claim: "ensuring it replaces a `loom-status` strand a pawl run's `lym loom start` put in the overview window first".
After card 3 the repository's module map contradicts itself and the code.

### [LOW] The one-overview comment sweep misses four statements in reed and orchcli

**Location:** `internal/reedengine/ownwindow.go:170`, `internal/orchcli/runs.go:1`, `internal/orchcli/runs.go:39`, `internal/reedcli/follow.go:38`

**Issue:** The decision record's scope takes in every package doc and comment in reed, orchcli and loomcli that states one overview strand per session.
The cards carry the doc.go paragraphs and most member comments, but four statements stay singular:
- the `isOverview` doc comment says it "places a strand in the session's overview window", and card 1 lists `isOverview` only under Uses;
- the `runs.go` file header and `runsStrandAddSpec`'s doc comment both call the runs window "the session's overview window", and card 3 edits `runsStrandAddSpec`'s code and never its comment;
- `lym reed follow`'s help says the relevant window is "the newest live agent window, else the overview", which D2 makes "else the lowest-index overview window", and no card lists `reedcli`.
This is one gap in how the sweep was enumerated, so it is filed once.

### [NIT] The relaunch row names a command the IfAbsent relaunch ignores

**Location:** `_lym/plan/01-reed-overview-order-key.md`, the `RelaunchingOrReplacingEitherOverviewKeepsTheOrder` bullet

**Issue:** The IfAbsent relaunch branch (`internal/reedengine/strand.go:624-631`) launches the strand's stored `ResumeCmd`, else its `Cmd`, and never the spec's.
`pawl` was added with cmd `exit`, so its relaunched pane exits again.
The order assertion is still reached, but the row describes a live relaunched pane that never exists.

## Focus settlements

- **Fidelity, D1 to D5 and the Constraints.**
  - D1 is carried by card 1, D2 by card 2, D3 by cards 3 and 4 (the constant in card 3, the add spec in card 4), D4 by card 3, and D5 by card 4's heal case and card 1's key-zero decode row.
  - Every Constraint is carried. reed's new refusal names "its spawner" rather than a beam or loom name. The overview-window properties are pinned by card 1's marks and current-window checks. Window order after the overviews is pinned by card 1's agent-after-both step. The two named constants are `orchcli.RunsStrandOrder` and `loomcli.statusStrandOrder`. Every placement and focus rule is pinned at tier0 over parsed listings.
  - The scope line's doc sweep is carried apart from F1 and F2. reed's "Overview anchor" and focus-rule paragraphs are carried by cards 1 and 2.
- **Acceptance criteria.**
  - Every D1 criterion has a test that reaches it: the two add orders, agent and shell order, replacement and relaunch keeping order, select-on-replace of the viewed overview, equal keys, and the order refusal.
  - D2's no-live-agent fallback is pinned at tier0 for both attach and follow. The live-agent case is pinned by the existing follow row and the attach rows.
  - D2's "viewer moves to the other overview" case is pinned at tier2 through removal. Its pane-exit half cannot occur: reconcile never kills a present own-window pane (`planReconcile`). Card 2 says so, and no test is owed.
- **Card order.** Card 3 creates `orchcli.RunsStrandOrder` before card 4 derives `statusStrandOrder` from it. Cards 3 and 4 depend only on card 1's `Display.Order`. Card 2's tier2 subtest follows card 1's subtests. No card depends on a later one.
- **Refusal reachability.**
  - The new `UpdateStrand` order refusal is reached by `UpdateStrand` with an overview display of another order, which `validateDisplay` and the anchor-move check pass. `TestUpdateStrandLocked`'s `OverviewOrderChangeRejected` row reaches it.
  - The removed `AddStrand` refusal goes with its only test. `ReplaceStrand`'s third-strand refusal reached only through `addStrandLocked`, so it goes with it.
- **Attack surface.** Card 1's Intent states the bound: `lym reed add` refuses the overview anchor, no shuttle caller sets it, and explicit-name uniqueness keeps each spawner to one copy.
- **Edit typing.** Re-sign arrows are present on `overviewWindowInsertArgs`, `agentWindowPanes` and `statusStrandAddSpec` (which loses its strands parameter). No other member's signature changes. Member glyphs are listed wherever an Intent names a symbol.
- **Test economy.** No card adds a test for behavior an existing test covers. New coverage lands as rows and subtests of the existing reed placement, attach, follow, update and loomcli status-strand tests.
- **Verify lines.** No card carries a `**Verify:**` line.

## Rejected

- `lym board get` with the bare slug was refused as invalid JSON during reading. That was a reading step, not a plan defect, and the board entry's scope is restated by the decision record.
