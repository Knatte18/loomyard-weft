---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed-differently
    reason: "Keeping the last child part is done as proposed, but with no last reading that would keep `child none` for a child that exists, so a stopped run would mail `child none` and then a second line for the same stop. A first reading over a newer child is therefore not kept at all (readWatchedRun returns ok false), and the row AFirstReadingOverAnOlderListingKeepsNone pins it."
  - id: F9
    disposition: fixed
  - id: F10
    disposition: disputed
    ground: premise false
    rebuttal: "No holder tree is ever read on Windows. gateHoldProcesses samples the tree before it scans for the marker, TreeSample returns errors.ErrUnsupported there, and the step returns before MarkedPIDs runs. The notice drops no tree it had read, and the proposed branch is reached on no platform the repository builds."
    evidence:
      - internal/proc/proc_windows.go:90
      - internal/proc/proc_windows.go:91
      - internal/dobbyengine/gatehold.go:152
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed
  - id: F20
    disposition: fixed
---

## Changes

- F1: `gateHoldFactorKey` and `gateHoldMinKey` are declared in `keynames.go` and used in `ConfigSpec` and both refusals.
  The comments and docs the finding lists name their keys in full, as do the ones the same diff added in `donerun.go`, `loop.go` and the `tasksetting_test.go` header.
- F2, F7: `TestDobbyGateHolders` and the land-notice test write config through `configengine.Set`.
  The first asserts every mapped holder field and provokes the load failure by removing `gate.yaml`;
  the second asserts `GateHolds.TaskWorktree`.
- F3, F13, F14, F16: rows and assertions in `TestGateHolds`.
- F4, F5, F12, F15, F17, F18: rows and assertions in `TestRunWatch`.
- F6: a `TestClearDoneRuns` case over a read-only record directory.
  It skips on Windows and as root.
- F8: a child written after its run was listed is not taken that tick.
- F9: `readNoticeChild` reports `PairDir` for a present worktree with no child status, `compared` ignores it, and the `ReadChild` doc says so.
- F11: the `cleanup` help and both `docs/overview.md` sentences name the run watch holds.
  The `batten run` and `beam create` help say the removal waits until the run's landing and run watch are settled.
- F19: `LoadConfig` refuses a NaN or infinite factor.
  `gateHoldBound` also caps a finite factor whose product passes the duration range, which gave the same defect.
- F20: the `Holder.Marker` doc and the `runMarkerEntry` doc state the nested-run bound.

The first commit of this round names F10 and carries its proposed branch;
the commit `dobbyengine: fix F10, F19` takes that branch, its doc sentences and its test row out again.

After the round, the gate flagged comment lines of `config.go`, `runwatch.go` and `tasksetting.go` that wrapped mid-sentence.
Those comments are rewritten as one sentence per line, in their own commit.

`lym gate test --tier 0 --tier 1` over `dobbyengine`, `battencli`, `beamcli`, `gateslot`, `gatecli` and `beamsetting` passed with no failure before that comment repair.
`lym gate test --tier 0 ./internal/dobbyengine` passed after it.

The verify run then failed `TestCLIForm_EveryCommand`:
the F11 sentences had pushed the `batten run` and `beam create` help past the byte cap.
Both are shortened in their own commit, and `lym gate test ./cmd/lym ./internal/battencli ./internal/beamcli` passed in tier0.

## Self-check

- fix-code-correctness 1, MEDIUM (the F8 row never reached the new branch): fixed as proposed.
- fix-consistency 1, LOW (`ReadChild` seam doc): fixed as proposed.
- fix-consistency 2, LOW (`runMarkerEntry` doc): fixed as proposed.
- fix-consistency 3, LOW (`StatusSeen` and `ChildSeen` doc): fixed as proposed.
- fix-proposal 1, MEDIUM (F10 premise false): fixed as proposed; F10 is disputed and its change reverted.
- fix-proposal 2, LOW (a finite factor still overflows the bound): fixed as proposed.
- fix-proposal 3, LOW (`ReadChild` seam doc): fixed as proposed, merged with fix-consistency 1.
- fix-affected-set 1, LOW (stale done-run removal sentences): fixed as proposed.
- fix-affected-set 2, LOW (`runMarkerEntry` doc): fixed as proposed, merged with fix-consistency 2.
- fix-affected-set 3, LOW (`ReadChild` seam doc): fixed as proposed, merged with fix-consistency 1.

## Disputed

- F10, `premise false`: Windows never reads a holder tree (`internal/proc/proc_windows.go:90-91`), and `gateHoldProcesses` returns at that failure (`internal/dobbyengine/gatehold.go:152`) before it reaches the marker scan.

## Deferred

## Rulings
