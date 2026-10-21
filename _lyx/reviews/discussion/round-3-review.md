---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:290
    summary: "Open risks says a card Verify line lacking --wait runs unslotted and fails at the keep-cache-warm cap, but D8 gives such a line the one-slot path, where its nested gate call inherits the slot and never meets the cap"
    fix: "In the second Open risks bullet, replace the clause saying a gate call lacking `--wait` runs unslotted and fails on a busy hub at the keep-cache-warm cap with D8's actual behavior: a card `**Verify:**` line that is not one `lyx gate test --wait` call, such as an in-flight `lyx gate test ./internal/x` with no `--wait`, takes the runner's single slot, its nested gate call inherits that slot, and it waits for the slot under the runner's 10-minute timeout without failing at the cap. Then limit the rewrite-and-rebaseline remedy to the `--tags` case, which D2 refuses, and drop the busy-hub failure claim."
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:261
    summary: D13 moves the -trimpath and -p flags from the shared Lease.Env into each non-tier holder, but no acceptance criterion pins them and the declared PATTERN-gate-slot text says only own Go flags
    fix: "Add one Acceptance criteria bullet after the `cmd/testtiming` bullet saying that each non-tier holder, a card `**Verify:**` line that is not one `lyx gate test` call whether Go re-run or `lyx gate card-verify` runs it, the `internal/planglyph` type load and every `cmd/testtiming` mode, passes its child GOFLAGS carrying `-trimpath`, plus `-p=<tier0 parallel>` when the hub declares `tier0` and no `-p` when it does not, and that `cmd/testtiming` outside a hub uses the CPU count with no slot. In the `## PATTERN changes` bullet for `PATTERN-gate-slot` and in the Goal-adjacent mirror of that text, change setting its own Go flags to setting its own `-trimpath` and `-p` Go flags, so the invariant keeps naming what the removed slot-wide flags guaranteed."
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:5
    summary: The Goal's first sentence attaches which runs numbered tiers to D13's non-tier slot holders instead of to lyx gate test
    fix: "Reword the first sentence of `## Goal` so the relative clause follows its subject: Every test run in a hub, an agent's own and every verify gate's, apart from D13's non-tier slot holders, goes through `lyx gate test`, which runs numbered tiers the hub declares in `gate.yaml` over a scope lyx computes, takes the gate slots each tier asks for, enforces each tier's timeout, counts its tests, and logs every tier run where it survives teardown."
---

### [LOW] Open risks misdescribes a Verify line lacking --wait

**Location:** `_lyx/discussion/decision-record.md:290`, against D8 at `_lyx/discussion/decision-record.md:179`-`180` and D2 at `_lyx/discussion/decision-record.md:83`.

**Issue:** D8 sends only a line that is one `lyx gate test --wait` call, with no shell control operator, down the unslotted path.
Any other line keeps the runner's single slot, so a line such as `lyx gate test ./internal/x` without `--wait` is slotted, its nested gate call inherits the slot (D6), and the runner's slot wait has no keep-cache-warm cap (`runCardVerify` in `internal/websterengine/cardverify.go` acquires under `context.Background()`).
The Open risks bullet says the opposite: that such a call runs unslotted and fails on a busy hub at the cap.
The plan writer reads Open risks as a statement of what the design does, so the bullet states a failure mode that D8 rules out and prescribes a rewrite-and-rebaseline step the line does not need.

### [LOW] Non-tier holders' Go flags have no acceptance criterion and a vague declared bound

**Location:** `_lyx/discussion/decision-record.md:261`, with `_lyx/discussion/decision-record.md:317`-`318` and the `PATTERN-gate-slot` bullet at `_lyx/discussion/decision-record.md:328`.

**Issue:** Today `Lease.Env` in `internal/gateslot/pool.go` appends `-p=<GoParallel> -trimpath` for every holder.
D13 empties it down to the inheritance variable, so the card Verify runner, `internal/planglyph` and every `cmd/testtiming` mode each have to set the flags themselves.
The acceptance criteria say the card line `go vet ./internal/x` holds one slot and that `cmd/testtiming` holds one slot in each mode, and neither says anything about the flags.
The declared `PATTERN-gate-slot` text, which binds the non-tier holders, says "setting its own Go flags", which a holder satisfies with no flag at all.
A holder that drops `-trimpath` or `-p` would pass every listed criterion and the declared text, while `PATTERN-trimpath-safe-root` and the machine's load cap depend on them.
The focus asked whether the record makes the card-line run hold one slot with `-p` and `-trimpath`: D8 and D13 do decide it, so the open part is only that the decision is unpinned.

### [NIT] Goal's first sentence attaches its relative clause to the wrong noun

**Location:** `_lyx/discussion/decision-record.md:5`

**Issue:** In "apart from D13's non-tier slot holders, which runs numbered tiers the hub declares", "which runs" reads as the non-tier holders running the tiers, and the verb is singular where the holders are plural.
The subject meant is `lyx gate test`.
The Goal is the first thing the plan writer reads, so the sentence should not need a second pass.

## Rejected

Each item below is a focus question settled with a verdict of not a defect, grounded in what I read.

- Focus 1, the fork's run of a card `**Verify:**` line: settled by the record, with F2 as the one residue.
  D8 sends the fork through `lyx gate card-verify`, which uses the Go re-run's runner, so the raw-go row's pass-through of a package-scoped `go vet` or `go build` never reaches a card line.
  A line that is not one `lyx gate test --wait` call holds one slot under the 10-minute timeout from the moment the slot is held (D8, D13).
  The declared `PATTERN-gate-slot` text names the non-tier holders with a one-slot bound, and `PATTERN-test-tiers` names the card line as an exception.
  An agent's own ad hoc package-scoped `go vet` outside a card line is today's raw-go guardrail and is not a card Verify run.
- Focus 2, `cmd/testtiming` modes: settled.
  D13 decides all three modes, each holding one slot for the whole run with the prebuild included, and states the premise correctly: `cmd/testtiming/resources.go` takes a slot only in `-resources`, and `cmd/testtiming/prebuild.go` holds one for the build alone, while the default `run` in `cmd/testtiming/main.go` takes none.
  The declared `PATTERN-gate-slot` text covers the slot through "one of the non-tier holders `internal/gateslot`'s doc lists", and the declared `PATTERN-test-tiers` text says `cmd/testtiming`'s runs in every mode, so both match D13.
  The prebuild's own slot acquire becoming part of the run's single hold is a restructuring the plan writer derives from D13's "prebuild included", and is not a record gap.
- Focus 3, the confirm-the-fix pass: settled.
  D8 says each failing-test run and the package pass is its own `lyx gate test --wait --tier <failing tier>` call, joined to the round's gate call with `&&`, which is the shape `publishFailureSteps` and the join in `internal/loomshed/verifygate.go` produce today.
  D8's claim that every command `verifytree.Verify` runs is one such call or such calls joined by `&&` agrees with it.
  The card and batch gates' `lyx loom lint-comments` and `lyx loom lint-pattern` run as fork steps and Go lints outside `verifytree.Verify`, so they do not break the claim.
- Focus 4, the `.md`-only acceptance criteria: settled.
  The criterion at line 303 names `go` for the empty-set no-command result and `none` for the whole run with a covered tier skipped, matching D3.
  The criterion at line 307 names `go`.
  Under `none` a computed scope is never empty, so D1 and D2's empty-scope clause and D3 do not conflict.
- Attack-surface and board-entry checks: no finding.
  `--wait`, `card-verify`, `slots: all`, `refuse_raw` and the `{args}` placeholder each state their bound or run inside an existing bound, and no Decision writes this task's own board entry.
