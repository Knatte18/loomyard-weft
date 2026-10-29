```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I looked for what reads, asserts or describes it.

**Per change, the set I found:**

1. **New stderr line on a passing `--card` call (`run.go:173`, `printNotRunTiers`).**
   - `failedRunsLine` (`internal/loomshed/verifygate.go:237`) matches only `gateslot.FailedRunsPrefix`. It reads failed calls only, and the new line prints only when `code == 0`. Covered, nothing to change.
   - `internal/loomshed/doc.go:53` describes the failed-runs closing line only. Unaffected.
   - No other caller or test parses a passing call's stderr. Covered.
2. **`internal/gatecli/doc.go` (3 lines added).** It states the line, its tiers (declared, minus listed, covered and manual-included) and its bound (output only, no other scope form). It matches D1 to D3. Covered.
3. **`contracts/specs/loom-plan-spec.md` Tier 1 bullet.** The added sentence matches the code and sits beside the existing batch/webster-gate sentence. Covered.
4. **`docs/overview.md:488`, the gate module row.** It describes the closing line of failed runs and `--card` behaviour, and the diff adds no mention of the new line. CLAUDE.md requires `docs/overview.md` only "if the module table or execution stack changes". A stderr line on one scope form changes neither, so this is not a finding.
5. **PATTERN.md and `pattern/`.** The directive declares no PATTERN changes. `PATTERN-gate-slot` covers verdict and records, which the diff leaves unchanged. Nothing is owed.
6. **Stencils and prompts.** I found none that restate the card gate's closing output or tell an agent to read it as green or not. The only affected-set hit is a doc, not a stencil. No stencil finding.
7. **Commit message and Pattern-Departure trailers.** The commit carries no trailer and the diff needs none. The commit message matches the diff.

**Finding:**

- **LOW, `consistency`.** `internal/gatecli/run.go:208-218` adds `printNotRunTiers`, which hand-rolls the tier-name join that `tierNames` (`internal/gatecli/scope.go:345-351`) already does with the same body. The diff duplicates an existing helper in the same package instead of using it. Evidence: `run.go:212-216` against `scope.go:345-351`.

**Tool calls:**
1. Grepped for the closing-line and `FailedRunsPrefix` consumers and for "tier0 only" in docs and PATTERN, because my context held no consumer list for the closing line.
2. Grepped `docs/overview.md` and `loomshed/doc.go` for closing-line text and read `verifygate.go:225-250`, because I had to confirm that `failedRunsLine` reads only the failed-runs prefix.
