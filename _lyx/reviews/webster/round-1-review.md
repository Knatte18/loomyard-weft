---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/creelengine/seam.go:87
    summary: ExternalSender forms the EXTERNAL agent name by string concatenation, which PATTERN-agent-name reserves to internal/agentname
    fix: >-
      In internal/creelengine/seam.go replace the name built by string concatenation in `ExternalSender` with the name from `agentname.FormatProgram(shortname, agentname.External)`.
      That call can fail on an invalid shortname, so change `ExternalSender` to return `(Sender, error)`.
      Propagate the error through `externalFromClaim` and its callers in `resolveSender` (internal/creelengine/identity.go lines 66-95),
      and update the callers in internal/creelengine/identity_test.go line 26 and internal/creelengine/verbs_test.go line 60.
      Add one `TestResolveSender` row or assertion for a refused invalid shortname, in the existing table and not as a new top-level test.
---

### [BLOCKING] EXTERNAL sender name is formed outside `internal/agentname`

**Location:** internal/creelengine/seam.go:87
**Issue:** `PATTERN-agent-name` says `internal/agentname` is the sole former of agent names, and card 1 added `FormatProgram` and `ParseProgram` for program-sender names, `EXTERNAL` included. `ExternalSender` still builds `<shortname>:EXTERNAL` itself with `shortname + ":" + agentname.External`. The same package parses these names through `agentname.ParseProgram` in `isExternalName` and `checkProgramClaim`, so a change to the name grammar would break the join and the parse separately. The entry carries no `(test)` mark, so review is the only check that catches it. The constant is correct and no behavior differs today. The violation is that the former is in the wrong package, and the rubric makes every listed-constraint violation BLOCKING.

## Rejected

Checked against the focus directive and found sound, so no finding:

- Import seam: no non-test file in `internal/creelengine` imports `reedengine`, `shuttleengine` or a package under it. `TestCreelStore_DeliverySeamImports` passes, and the engine names no tmux command.
- Sender identity: `resolveSender` in `internal/creelengine/identity.go` matches card 8 on every branch. A claimed strand name disagreeing with the pane, an outside peer claiming a strand, and a program claim from a non-orch pane are all refused. An unreadable strand list or an unknown pane pid refuses a peer in no known pane. Every privileged verb (`checkMayAct`, `Hold`) tests the resolved `Sender`, never the claim.
- Journal fold and retention: the fold is idempotent by id, a `sending` without an outcome folds as unconfirmed, ids continue past the high-water record, and carried records are synced before the old files are deleted. Holds, TTL (`aged`) and reminder state replay consistently with the live pool.
- Constants: retention, TTLs, hold cap, probe, sweep and reminder backoffs, and the body and reminder caps are named constants in `creelengine` with the stated values and are used at each enforcement site. No config file is read.
- Recordless strand: `Runner.Send` and `Runner.Interrupt` only drop the refusal on `ErrNoRunRecord` plus a live reed strand. They keep the pane-only readiness reading, the pre-write guards, no pending-send record and no clearing of unlanded text. An unreadable run directory still refuses. The rows card 5 lists exist in `internal/shuttleengine/run_test.go`.
- Delivery adapter: `internal/creelcli/delivery.go` maps the three shuttle sentinels. The engine asks `Idle` before every `Send`, and `Send` is itself guarded.
- Per-card mechanical checks: every card here is Create or Edit, with no Rename or Delete groups. `lyx gate test` passes for the creel packages, and with `--tags integration` for `internal/creelcli` and `cmd/lyx`, which holds the test-budget and daemon-wakeups scans.
- Test economy: the top-level test count per package matches the budget rows and the test list on each card. No duplicate or per-helper test and no uncovered behavior turned up.
- `PATTERN.md` cuts: the entries shortened by card 11 keep their cut detail in their background files.
