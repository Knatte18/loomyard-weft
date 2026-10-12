# Support log — run-robustness

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry and the entries touching it: `board-field-drop` (board.json keeping, in Merge-Bouncer), `verify-watchdog` (Windows process-tree kill), `proc-child-lifetime` (fold gate test's child lifetime into `internal/proc`), `go-driven-shed`, `lyx-halt`.
2. Found the batten halt's source: `state.ReadJSONStrict` sets `DisallowUnknownFields`, and every shed `status.json` reader uses it; `shedrun.decodeSeed` and `battencli.judgeStatus` decode strictly by hand.
   `routed_to` lives in `HistoryEntry`, so keeping must reach nested objects.
3. Found verify-2's source: `internal/websterengine/verifygate.go` runs the verify command a second time in the same evaluation after any failure other than `res.TimedOut`; an orch SIGQUIT makes go test exit 1, which reads as an ordinary failure.
   `internal/loomshed/verifygate.go` does not rerun.
4. Settled a generic merge in `internal/state` (D2) over per-type extra fields, array elements matched by index only while unchanged (D3), and a CPU watchdog in `lyx gate test` (D7) over a lower go test `-timeout`.
5. Lens review (premises, goal-scope, board-fit, pattern-fit, scenarios, leanness); every fork returned.
   - leanness: LegacyParent call hidden in Notes — fixed, moved into D4.
   - leanness: perishable record-type list in D2 — fixed.
   - leanness: D7/Constraints repeat of hub-wide config — fixed, kept in Constraints.
   - leanness: no-op `PATTERN-sole-parsers` constraint — fixed, cut.
   - leanness: meta line in Auto-mode assumptions — fixed, cut.
   - leanness: incident timeline in D5 — fixed, trimmed to the premise.
   - leanness, goal-scope, pattern-fit (merged): unbounded non-merging board write — fixed by removal after board-fit showed it unneeded.
   - board-fit: `board-field-drop` landed at `78e31aa06` with `storeFile` implementing `json.Marshaler`, so D2's opaque rule already covers board.json — fixed: opt-out dropped, baseline named.
   - board-fit: D9 conflicts with read-only decodes outside `state` (`runs-overview`, batten cleanup) — fixed, D9 narrowed to "no refusal; rewrites keep".
   - board-fit: name `lyx-halt`, `proc-child-lifetime`, `verify-watchdog`, `dotnet-scoped-verify` — fixed.
   - goal-scope: D2 reaches non-run records — fixed, named in Scope with the extra-read cost in the bound.
   - goal-scope: busy non-test process and non-gate verify commands lack rationale — fixed in Scope Out.
   - goal-scope: `ReadJSONStrict` rename is churn — rejected: the name would state behavior it no longer has.
   - goal-scope: docs of `proc`, `shedrun`, `battencli` — fixed in Scope.
   - premises: -1 exit from a signal-killed shell missed — fixed in D5.
   - premises: `/proc/<pid>/stat` comm truncates `.test` names — fixed in D7, Notes and an acceptance criterion.
   - premises, board-fit (merged): watchdog only reaches `lyx gate test` runs — fixed in the D7 bound and Scope Out.
   - premises: "every shed status.json reader" overstated — fixed to "every standalone reader".
   - premises: parentreview's raw reader — fixed as a Note to confirm.
   - premises: older binaries tolerate the new gate.yaml key — fixed as a Note.
   - pattern-fit: refusal-spec row for the changed stuck reason — fixed in D6 and an acceptance criterion.
   - pattern-fit, scenarios (merged): integration test waits out the 5 s period — fixed: period injectable, test uses 1 s.
   - pattern-fit: PATTERN headroom — fixed: one line plus a background file.
   - pattern-fit: `LocalKeys` membership and 0 accepted — fixed in D7.
   - pattern-fit: hedged daemon-wakeups listing — fixed: listed unconditionally.
   - scenarios: absent key silently disables the watchdog — fixed: absent reads as 120, negative refused.
   - scenarios: false hang match unbounded — fixed in the D5 bound.
   - scenarios: `-bench`/`-fuzz` runs quit by design — fixed: exempt.
   - scenarios: false quit cascades to stuck — fixed as an Open risk.
   - scenarios: missing criteria for hanging rerun, once-only, non-Linux — fixed; nested gate test double SIGQUIT fixed by no watchdog in an inherited slot.
   - scenarios: map versus array update rule — fixed: D3 says why a map key needs no check.
   - scenarios: fresh record inherits old file's keys — fixed as an Open risk.

## Rejected alternatives

- Per-type extra-field maps with custom `MarshalJSON`/`UnmarshalJSON`, as `board-field-drop` does for the board: one implementation per record type and per nested type, and a new record type would have to remember it.
- Keeping only top-level unknown keys: the incident's `routed_to` is inside a history entry.
- Matching array elements purely by index: a rewritten or reordered array would carry kept keys onto a different element.
- A schema-version refusal on run records: it halts the run, which is the bug.
- `encoding/json/v2`'s unknown-member field: still behind `GOEXPERIMENT=jsonv2` in Go 1.26.
- Lowering go test's `-timeout` in `lyx gate test`: a wall-clock limit fails slow but healthy packages and does not target the CPU load the entry names.
- Watchdog in `internal/verifyrun`: it is language-agnostic, and SIGQUIT means stacks only to a Go binary.
- A separate non-merging `state` write for `board.json`: unneeded once `board-field-drop` landed, since `storeFile` owns its encoding.
- Applying D3's unchanged-element rule to map values: a map key is an identity, so a stale-element mismatch cannot happen there.
- Classifying a hang from the exit code alone: go test exits 1 after a SIGQUIT test binary, so the log markers are needed.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): generic keeping in `internal/state` versus per-type? Generic; one mechanism covers every run record.
- Q2 (auto-pick): rename `ReadJSONStrict` once it is tolerant? Yes; "Strict" would describe behavior it no longer has.
- Q3 (auto-pick): YAML Go-written records in scope? No; tolerant by default and written once per attempt (assumption in the record).
- Q4 (auto-pick): agent-written contract files stay strict? Yes; strictness there catches agent error.
- Q5 (auto-pick): watchdog budget and threshold? One full core for 120 s, sampled every 5 s, `gate.yaml` key `busy_quit_sec`, absent reads as 120, 0 disables.
- Q9 (auto-pick): `-bench`/`-fuzz` runs and nested gate runs under the watchdog? Exempt both.
- Q6 (auto-pick): Windows watchdog? None; no SIGQUIT there.
- Q7 (auto-pick): a hang counts against the gate's attempts? Yes, through the normal re-prompt to the fixer.
- Q8 (auto-pick): new PATTERN entry with a scan? Yes; the failure class recurs with every field addition.
