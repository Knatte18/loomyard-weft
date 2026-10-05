---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomcli/circling.go:88
    summary: circling verb refusals that forward an error (status read, Bouncer lookup, record I/O) name no way forward and have no refusal-spec row or test
  - id: F2
    severity: MEDIUM
    class: scope
    location: contracts/specs/refusal-spec.md:178
    summary: decision add refusals for a failed status read and a failed append name a retry way forward in code and tests but have no loom section row
  - id: F3
    severity: LOW
    class: consistency
    location: internal/loomcli/circling.go:41
    summary: gofmt flags circling.go, decision.go and circling_test.go (missing space after the const assignment and after a table-row comma)
---

Sweep method: every `refuse(` in `internal/loomcli` (`circling.go`, `decision.go`), every Stuck `Reason` and `degrade` message the diff adds in `internal/shedadapters`, and the one config refusal.
Each was compared with its refusal-spec row, its asserting test and, where a cell quotes the message, the string in code.

Settled with no defect:
- `shedadapters` Stuck Reasons for an unreadable escalation record, an unreadable circling decision and an already-settled accept each name a way forward, have a shed-section row, and are reached by an untagged test asserting `way forward:` and the path.
  `escalate` forwards every `readEscalation` error, malformed or read I/O alike, through the same Reason, so no shared-helper error escapes without a way forward there.
- `circling: escalation record malformed` forwards `ErrEscalationMalformed` with `circlingWayMalformed`; row, code and test substring agree.
- `config: review key below 1` message ends with the row's quoted clause, and the config test asserts it.
- `decision add` flag, `--by`, no-record, Discussion-Write-running, no-heading, check-finding and commit-failed refusals each name a way forward, match their rows and are asserted.
- Delete groups (cards 01 and 15): a repository grep for `IsLedgerPath`, `ReadLedger`, `shedadapters.Ledger`, `AnomalyRecurringFinding`, `recurring-finding`, `detectRecurringFinding` and `LedgerObservation` finds no reference outside `_lyx`.

### [MEDIUM] Circling verb refusals that forward a helper error carry no way forward
**Location:** `internal/loomcli/circling.go:63`, `:73`, `:88`
**Issue:** `circlingVerb` formats three refusals as bare `read the run status: %s` and `%s`.
The last one forwards every `RecordCirclingDecision` error that is not one of the three sentinels: a `readEscalation` read I/O failure, a `ResolveRound` failure, a failed create or write of the decision file.
Card 8 widened what reaches that site (`readEscalation` now runs first), and the diff touched the switch.
`PATTERN-refusal-way-forward` requires every refusal reachable through a loom verb to name its way forward and have a section row plus a reaching test; none of the three has a row, and no test reaches the generic branch.
`decision.go` already handles the same shape with `decisionWayRetry`.
**Fix:** add `circlingWayRetry` (re-run the verb; nothing is recorded, or the existing decision file stays, naming the file where the error carries it), append it at all three sites, add the matching loom rows, and add test rows that reach the status-read, Bouncer-lookup and record-error branches and assert `way forward:`.

### [MEDIUM] Missing spec rows for the decision add retry refusals
**Location:** `contracts/specs/refusal-spec.md` (loom section, `decision:` rows)
**Issue:** `decisionVerb` refuses on a failed status read (`decision.go:83`) and on any other append error (`decision.go:103`) with `decisionWayRetry`, and `decision_test.go` asserts both, but the `decision:` rows list neither.
The pattern requires the row in the same commit as the refusal.
**Fix:** add `decision: status read failed` and `decision: append failed` rows to the loom section, with the way forward quoting `re-run "lyx loom decision add"; nothing is appended`.

### [LOW] gofmt violations in touched files
**Location:** `internal/loomcli/circling.go:41`, `internal/loomcli/decision.go:44`, `internal/loomcli/circling_test.go:78`
**Issue:** `circlingWayDecided   =` and `decisionWayFindings  =` lack the space before the raw string, and the `"second decision",circlingFake` table row lacks a space after the comma; `gofmt -l` lists all three files, and none of them is unformatted on the base.
**Fix:** run `gofmt -w` on the three files.
