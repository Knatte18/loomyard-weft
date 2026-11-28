```yaml
lens: code-affected-set
outcome: reported
```

Scope: the code-affected-set lens. For each thing the diff changes, I found what calls, reads, asserts or documents it and checked whether the diff leaves any of it wrong.

**Per change: what reads, calls or documents it, and whether it is covered**

1. `reedengine.StrandStatus` gains `Down` (`internal/reedengine/lifecycle.go:87`), and `downCause` with four `DownCause` constants is added (`internal/reedengine/apply.go:79-111`).
   - **Readers of `StrandStatus`:** `orchroute.ReadingOf` reads `Down`; `trackedStrand`, `statusFields` and `restartWatcher` in orchcli; `orchStrands`, `soleOrchStrand`, `relaunchStrand` and `adoptListedStrand` in orchcli `start.go`; orchcli `runs.go`; shuttleengine `stop.go`, `attach.go` and `liveStrand` in `run.go`. Only `ReadingOf` reads `Down`. The other readers are additive-safe: the fields they read keep their meaning.
   - **`lym reed status` JSON** (`internal/reedcli/status.go:45-52`): it emits `elsewhere` but not `down`. The record asks for the cause in orch status and the routing's log line only, so this is no finding.
   - **Package doc** `internal/reedengine/doc.go:87`: updated. It agrees with the predicate: "listed in this session" covers both pane-unlisted and pane-elsewhere.
   - **Tests:** `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing` gains `wantDown`. Covered.
2. orchroute: `liveFrom` becomes `readingFrom`, with `Reading`, `ReadingOf` and three causes added (`internal/orchroute/live.go`).
   - `liveFrom` has no callers left; a repo-wide search finds no leftover reference.
   - `ReadingOf` is called from `internal/orchcli/status.go:220` and from `readingFrom`.
   - **Package doc** `internal/orchroute/doc.go:3`: updated. File header `live.go:1-2`: updated. Test renamed to `TestReadingFrom`.
3. `orchroute.Live` keeps its signature and gains a log line.
   - **Callers:** orchcli `OrchLive`, then on to beamcli `orchview.go:30`, loomcli `decision.go:1661` and `parentreview.go:320`, and orchcli `NearestLiveParent` and `NoticeDestination`. Those two reach many wiring packages through the `orchcli.Notify*` routing. Creel calls it at `internal/creelcli/cli.go:72`, through `Asker`.
   - Signatures are unchanged, so no caller needs an edit.
   - The doc of `OrchLive` (`internal/orchcli/live.go:65`) is updated.
4. `lym orch status` gains the `strand_down` key and help text (`internal/orchcli/status.go:172-174, 220`).
   - **Tests:** `TestStatus_Envelope` gains rows. `smoke_cycle_test.go:239` and `smoke_live_test.go:141` read only `strand_live`, which is unchanged.
   - **Docs and stencils:** no repository doc, stencil or spec describes the `lym orch status` fields; the search of `docs` and the stencil directories found none.
   - **Help list:** it omits `no-reed-session`. That is correct, because the verb returns an error when `Strands()` fails before `ReadingOf` is reached. The help list matches what the verb can emit.
   - **The `statusFields` doc** (`status.go:36-37`) lists the envelope's fields and does not mention `strand_down`. That is no defect, because the key is added outside `statusFields`.
5. PATTERN and commit trailers.
   - The directive declares no PATTERN changes, and the range carries no `Pattern-Departure:` trailer.
   - PATTERN-code-conventions "Named constants" applies to the new cause names. See finding 1.

**Findings**

1. **BLOCKING · design · `internal/orchcli/status.go:172-174`**
   The `strand_down` help text spells the six cause values as string literals: `no-strand-recorded`, `strand-not-in-reed-state`, `pane-dead`, `pane-unlisted`, `pane-elsewhere` and `start-shell`. Named constants for each are declared in `internal/orchroute/live.go:20-26` and `internal/reedengine/apply.go:84-92`.
   PATTERN-code-conventions ("a name or path that can go stale: one named constant, declared once and referenced everywhere, never a repeated literal") forbids that repetition.
   Renaming a cause would silently leave the help wrong. The same help string already builds config key names from code (`orchKeyName(...)`, `status.go:154`), so building it from `orchroute.DownNoStrandRecorded`, `string(reedengine.DownPaneDead)` and the rest fits the pattern.
   **Evidence:** `status.go:172-174`; `live.go:20-26`; `apply.go:84-92`; `pattern/PATTERN-code-conventions.md` "Named constants".
2. **LOW · consistency · `internal/orchroute/live.go:66,68`**
   The log message `"orch: the orch reads as down"` is repeated as a literal in the Debug and Info branches.
   The change description (`_lym/landing/summary.md`, "To check by hand") names this line as the one an operator searches for. Two literals can drift apart, so the line should be one named constant.
   **Evidence:** `internal/orchroute/live.go:66` and `:68`, in the diff `002-internal-orchroute.diff`.

**Tool calls**

1. A grep across the repo for `liveFrom`, `strand_elsewhere` and `strand_live`, plus the non-reed consumers of `StrandStatus` and reedcli's `Elsewhere`. My context had no leftover references to the old name or the other readers of the struct and envelope.
2. `internal/reedcli/status.go:30-70`, plus a search of `docs` and the stencils for orch status fields. My context lacked reed status's JSON shape and any doc describing the envelope.
3. `internal/reedengine/doc.go:72-90`, plus grep of `PATTERN-reed-panes.md`, `PATTERN.md` and `PATTERN-agent-addressing.md` for liveness rules. My context had the doc.go hunk without its surrounding paragraph, and no PATTERN background on liveness.
