---
verdict: APPROVED
---

## Alignment at pr-gate: engine-usage-readers

PR 667 at `ebc6a060cafafdad930fa9462a8b9194b7fd0d77` was compared against `main` (merge base `c7ea67f83`, the current `main` tip) and against the board entry and the decision record.

### Goal and decisions

The diff matches the board entry's four scope points and the recorded decisions.

- Neutral types and capabilities (`LocatedSession`, `SessionRead`, `MessageUsage`, `TranscriptReader`, `MessagePricer`, `RecordReads`) are in `internal/shuttleengine/transcriptread.go` (D1, D2, D3).
- `claudeengine` implements the locator, the reader and pricing, and `SessionUsage` shares one usage parse with the reader in `usage.go` (D4, D5).
- `shuttleengines.RecordReaders()` hands out one adapter per capability, built with a zero config, and `PATTERN-shuttle-provider-seam` gains the matching line (D6).
- tokencount reads every run and the orch through the read set, the `-projects` flag and the `~/.claude/projects` resolution are gone, and its non-test Go holds no Claude transcript field, `.claude` path or price figure (Scope, acceptance criteria).
- The header names engines without a transcript reader (D7), the `-no-cutoff` note and flag help name located sessions, and the default selection's no-runs error names the engines asked (D10).
- Calibration takes instruction files from the engine that located the run's Webster session, read at the base commit (D8).

None of the work falls under the decision record's Out list.
The changes beyond tokencount and the engine packages are the shuttlefake golden of neutral reads, which feeds tokencount's fake-driven tests (D6), and a one-line test-budget bump.

### Open risks carried over

- `engine-model-choice` landed before the merge base (`371df4569`), and the live `websterengine.MerriamBase` takes the engine's instruction-file list, as calibration now does.
  The first open risk is therefore settled for this landing.
- The other two open risks, Codex's shape fit and the Claude-shaped weight coefficients, are deferred to `codex-engine` by the record itself, and neither blocks this gate.

### Overlap

- Peer `webster-deviations-list` (row 13, Webster-Bouncer): it shares only `cmd/lyx/testdata/test-budget.yaml` with this run, at different rows.
  This is a shared bookkeeping file, not an overlap.
  For the landing-order record, this run (own_row 22) is further along, so it lands first and the peer takes the budget file at Merge-In.
- No parent commits are listed.

### Superseded

This run makes no board entry obsolete.
`creel-stage1-fixes` depends on it, and `codex-engine` builds on it without being replaced.
