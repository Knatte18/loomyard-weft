```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the affected set:**

1. **New `restoreStatusPane` seam, wired to `reedEngine.Up` + `ensureStatusStrand`** (`cli.go`, `wiring.go`).
   - Set: `ensureStatusStrand`'s doc comment, the `sharedbootstrap.go` header and its log messages, `start.go` step 4 (the other caller), and the docs on who calls it.
   - Not covered: the header and doc comment still say only `start` calls it (finding 1).
2. **Resume restores the pane on both wake paths** (`resume.go`).
   - Set: the `resume.go` header, the `resumeCmd` Short and Long text, the `doc.go` `# Resume` section, `docs/overview.md` lines 1210-1218 (loom passage), lines 1183 and 2040-2042 (start's bootstrap passages, still true), and `smoke_resumearming_test.go` (doc and message updated, still true).
   - Covered except the Short (finding 2).
3. **New envelope field `status_pane`** (`resume.go`).
   - Set: `doc.go:112`, the `resumeCmd` Long, `resumeEnvelope` in `resume_test.go`, and the summary. No spec in `contracts/` lists resume's envelope fields; `grep status_pane` finds nothing outside the diff.
   - Claim wrong for the replace branch of the callee (finding 3).
4. **Resume can now move the viewer's window** (the replace through `ReplaceStrand` selects the new overview window).
   - Set: the `smoke_drivingsurface_test.go` subtest, the Loosened-Check trailer, the summary's design decision, the `resume.go` header and the Long's "never switches a tmux client".
   - The header and Long say "tmux client", which a select-window doesn't contradict. The summary's "never switches a viewer's window" is false (finding 4).
5. **Commit messages and departures.**
   - The Loosened-Check trailer gives an observed measurement (@3 vs @1). The Root-Cause trailer names the defect the diff removes. No `Pattern-Departure:` trailer.
   - Covered.
6. **Declared PATTERN changes:** none declared, so nothing to land.
   - Repo rules touched: PATTERN-refusal-way-forward doesn't apply (`status_pane` is a warning on a success envelope, not a refusal). PATTERN-config-key-names doesn't apply. PATTERN-test-only-code is met (the fake lives in `_test.go`).
   - Covered.
7. **Stencils:** none changed. `contracts/stencils` and `contracts/orchrole` mention `lym loom resume` only as a way forward, with no claim about reed or the strand.
   - Covered.

**Findings:**

1. **LOW, consistency**, `internal/loomcli/sharedbootstrap.go:1,6,334-336,379`. The file header ("bootstrap steps that `start` and `step` share"; "ensureStatusStrand is `start`'s alone") and the doc comment ("Only `start` calls it"; "ends before the run-lock probe") are falsified now that resume calls it through `wiring.go:1384-1389`. Resume calls it after its own run-lock probe, at `resume.go:100` before `:147` and `:174`. The Warn at `:379` says "the next start will replace it", but a resume replaces it too.
2. **NIT, consistency**, `internal/loomcli/resume.go:276`. The `resumeCmd` Short "wake this task's parked loom driver after a halt, without starting anything" contradicts the new Long, which brings the tmux session up and adds a status strand.
3. **MEDIUM, design**, `internal/loomcli/sharedbootstrap.go:366-369`, as reached from `wiring.go:1388`. On a failed `ReplaceStrand`, `ensureStatusStrand` logs and returns nil, so `restoreStatusPaneBeforeWake` returns "" and the envelope carries no `status_pane`.
   - After a reboot, `Up` clears every pane binding (`reedengine/lifecycle.go:725-729`). The status strand is then found not live, and `resolveStatusStrandAction` picks replace (`bootstrap.go:231-232`).
   - So the entry's own situation runs the silent branch. That falsifies `doc.go:112`, the Long's "a restore that fails never fails the resume, and the envelope's "status_pane" names the failure", and the summary's matching claim.
4. **LOW, consistency**, `_lym/landing/summary.md`, Design decisions ("so it still never switches a viewer's window"). The range's own Loosened-Check trailer measures the opposite: current window after the resume = @3, want @1. `ReplaceStrand` selects the new overview window when it ends the viewer's window (`reedengine/replace.go:93-95`), and `statusStrandAddSpec` is overview-anchored (`bootstrap.go:36`).

Tool calls:
1. Grepped `contracts/` and stencils for statements about resume and reed, strand or pane; the manifest had no docs beyond the diff.
2. Grepped `webster-spec.md`, loomcli and `refusal-spec.md` for resume claims and envelope specs, to find documents that read the changed behaviour.
3. Grepped `docs/overview.md` and `contracts/` for status-strand statements and read the `doc.go` Resume section, to sweep the docs outside the diff hunks.
4. Read `docs/overview.md` around line 2042 and `loom-status-spec.md`, to check the bootstrap passage the previous grep turned up.
5. Read the `smoke_drivingsurface_test.go` header and the test's doc, to check its "never replace a status strand" claim against the new resume subtest.
