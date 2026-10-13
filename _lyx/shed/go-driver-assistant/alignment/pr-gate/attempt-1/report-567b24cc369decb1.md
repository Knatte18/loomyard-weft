---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: internal/loomcli/cli.go:492
    summary: run-window-bugs changes internal/loomcli and internal/battencli, and its card 6 edits the same Command AddCommand line and internal/loomcli/doc.go; no record gives the landing order between the two runs
    fix: The parent records the landing order between go-driver-assistant and run-window-bugs, for example as a decision in either run's decision record; under the rule applied to judge-cap-call, go-driver-assistant at PR-Gate lands first and run-window-bugs takes its internal/loomcli and internal/battencli changes at its own Merge-In.
---

### [BLOCKING] Package and file overlap with run-window-bugs has no recorded landing order

**Location:** `internal/loomcli/cli.go:492`

**Issue:** `lyx loom peers` lists `run-window-bugs` (Webster) as changing `internal/loomcli/checklist.go` and `internal/battencli/overview.go`, in two Go packages this run also changes (`internal/loomcli` heavily, `internal/battencli` in `arm.go`, `cli.go` and `wire.go`).
Its plan card 6 (`06-loom-fmt-verb.md`), not yet built, edits `internal/loomcli#Command`, the one-line `parent.AddCommand(...)` list where this run appends `c.driveCmd()`, and `internal/loomcli/doc.go`, which this run extends with the Go-driver and tackler sections.
The two designs do not conflict: the peer's loomcli and battencli edits are a status-pane `Resized` field and a new `lyx loom fmt` verb, and its record only expects an overlap with this run in `shuttleengine`, which this run does not change.
Neither run's decision record nor the board says which lands first; the recorded landing-order decisions in this run's record cover only `judge-cap-call`, which has landed and been merged in (`363355a49`).

## Scope

The run matches the board entry's goal and the decision record:

- Per-task choice: `driver_kind` in loom config (`internal/loomengine/config.go`, `template.yaml`), classified, open to the local layer, value-checked through `configreg`'s new `CheckValues`; batten's `childDriverOf`, `ChildDriver` and `refuseAdoptedSeed` follow D1, with the new refusal-spec row.
- Go driver: hidden `lyx loom drive` (`drive.go`, `driverules.go`, `drivehandback.go`, `drivewiring.go`) with its driver lock in `shedrun`; the rule table in `decideDrive` follows D3's rows in order and branches on envelope fields, kinds and facts only.
- Tackler: stencil `contracts/stencils/loom/loom-template-tackler.md`, call, consume, prompt and attempt records in `internal/loomcli` and `internal/loomengine/tackler.go`, the three config keys, and the in-flight marker read by batten's notice hold.
- Arbiter: the case-brief marker and the check-the-facts text in its stencil; the attempt records the brief path.
  Eligibility (`arbitereligibility.go`) is untouched, as the Out list requires.
- `lyx loom resume` spawns a Go driver for a halted, driverless go-seeded run (D10), with its refusal-spec rows updated.
- Batten: binary re-step (`innerrun_binaryrestep.go`), driver-lock liveness, tackler hold, and the spent-retry read from settle records (D4, D5).
- The orch role stencil gains the one D15 line.
- Shed sets `pause requested` and `cancelled` reasons on the empty-reason `paused` stops (D13).
- `shedverbs` exports `RunUntilStop` and `RunGoto` as the verb bodies, with no behaviour change, per `PATTERN-shed-verb-set`.
- `PATTERN-driver-choice-single-site`'s background and index line follow the new readers, as the constraints require.

Nothing on the Out list changed: the LLM driver's stencil, its guide and its fork path are untouched; `driverprompt.go` only moves the batten-watched marker parse into a shared reader that also returns the prime's scratch directory D4 needs.
The Webster-Review carry-overs F1 and F2 are fixed in `fbb38058d` and `75f6634a4`.

## Other overlaps, not escalated

- `run-window-bugs` also changes `PATTERN.md` and `docs/overview.md`, but a different PATTERN entry (`PATTERN-role-skills-typed`) and a different overview paragraph (the reed skills line); `cmd/lyx/testdata/test-budget.yaml` is shared bookkeeping.
- `logging-cleanup` (awaiting at Discussion-Bouncer, no changed paths) plans edits to `internal/battenshed/innerrun.go`, the orch role stencil's `## After landing` section and `contracts/stencils/claims_test.go`, all files this run changes.
  Its decision record names the `innerrun.go` overlap with this run and settles it: whichever lands second merges against the other.
- No parent commits are listed.

## Superseded

No board entry is made obsolete by this run: `arbiter-when`, `orch-split`, `report-timing` and `arbiter-fixes` each keep work this run leaves out.
