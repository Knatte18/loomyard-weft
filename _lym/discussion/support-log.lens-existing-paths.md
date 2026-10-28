```yaml
lens: existing-paths
outcome: reported
```

**Lens: existing-paths.** I traced the code paths that read or write what each decision changes. Two decisions break on an existing path and one plan note does not fit the code.

**Findings**

1. **MEDIUM — D5 (`//lym:seam` marker placement).** A seam marker stacked under a `//lym:guard` line breaks the guard scan, and the record does not address this.
   - Evidence: `internal/impactset/guards.go` `onlyKeepLinesAfter` allows only `//testtiming:keep` lines between `//lym:guard` and its `func Test…` line.
   - Anything else there raises `GuardScanError{Kind: GuardMisplaced}`, which is a hard scan error at the gate's guard step.
   - D5 says the marker sits "on the line above it, the way `//lym:guard` sits". A test that is both a guard and a seam test then fails the guard scan if `//lym:seam` sits between `//lym:guard` and `func Test…`.
   - D5's own parser has the mirror problem: it must allow guard and keep lines between the seam marker and the func.
   - `cmd/testtiming/keep.go` already lets `//lym:` lines stack between a keep line and the func, so testtiming survives.
   - Fix: D5 should either (a) have the guard scan admit `//lym:seam` lines, or (b) pin a stacking order, and say that its parser admits `//lym:guard` and `//testtiming:keep` lines.

2. **MEDIUM — D3/D4 exemptions.** A citation the writer cannot edit can fail the gate permanently.
   - D3/D4 skip `### Added after Discussion` entries and carry-over blocks.
   - They do not skip `## PATTERN changes` bullets that carry the ` — added after Discussion (<by>, <date>)` suffix. The spec says those are also written only by `lym loom decision add`, and a writer never edits them.
   - `discussionparser.AppendDecision` writes such a bullet when a decision is recorded during Discussion review. That bullet often cites a background file the task creates (`pattern/PATTERN-x.md`).
   - The next Discussion-Burler-Fix gate (`ValidateLoomDiscussion`) would then report `citation-path-missing` on text the fixer may not touch. The gate can only exhaust.
   - Fix: exempt the suffixed bullets too.

3. **LOW — D9 and the plan-writer note "D9 fits in `planGatePass`".** The note does not fit the existing callers or arguments.
   - `planGatePass` is also called by `planglyph`'s `Validate` (`internal/planglyph/planglyph.go:96`), which is the `--require-approved` path. Placed there, D6 and D8 also run in `--require-approved`, which D9's mode list leaves out.
   - `planGatePass(plan, worktreeRoot, loader)` gets no tier list. The `gate.yaml` tiers live in `loomshed.PlanContext`, which the verb leaves empty under `--require-approved`.
   - So D7 cannot sit in `planGatePass` as the note suggests. Under `--require-approved`, "no `gate.yaml`, skip" would be confused with "context not loaded".
   - Fix: state that D7 runs on the `PlanContext` side, like `rename-tool-missing`, and state whether `--require-approved` runs D6 to D8.

**Paths traced**

- **D1/D2**
  - `discussionparser.AppendDecision`/`AppendDecisions` run `Validate` only, so they survive and never see D1/D2.
  - `ValidateDarnDiscussion` returns early on `Validate` findings. Adding D1/D2 beside it survives.
  - The darn and loom gates in `shedrecipe/entries_gate.go` `resolvePlainGateClosure` call the shared functions, so they survive.
  - Added-after headings pass D1's exemption, so that survives.
- **D3/D4**
  - The loom discussion gate is built in `resolvePlainGateClosure`'s `"discussion"` case with paths only. `shedrecipe.Env` already holds `PlanIndex` and `WorktreeRoot`, so wiring needs only new `requireSeam`/`requireAbsRoot` calls and no new Env field. Survives.
  - The verb's loom branch (`internal/loomcli/validate.go:72`) survives once it passes `c.env.PlanIndex`.
  - The carry-over writer survives because of the exemption.
  - Bullets appended by `AppendDecision` break (finding 2).
- **D5**
  - impactset guard scan: breaks (finding 1).
  - testtiming keep scan: survives.
  - `commentlint`'s `directivePattern` already treats `lym:` lines as directives, so it survives.
- **D6–D9**
  - `loomshed` plan gate via `index.ValidateFormat` with `DoneCards`: survives.
  - `ValidateFormatAfter` (`PendingPlan`) skips done cards: survives.
  - `ValidateDispatch` callers (webster `beginbatch.go`, `runlevel.go`, `webstercli/validate.go`) never call `planGatePass`: survives as D9 states.
  - `planglyph.Validate` (`--require-approved`) and the tier list: mismatch (finding 3).

**Tool calls**

1. Grepped the callers of `ValidateLoomDiscussion`, `ValidateDarnDiscussion` and `discussionparser.Validate`. My context had only `gates.go` and `validate.go`.
2. Grepped the gate constructors and `AppendDecision`'s `Validate` use, to find which rows build the gate and whether decision add runs the new checks.
3. Read `shedrecipe/entries_gate.go`'s discussion and plan gate cases, to see what Env provides to the gates.
4. Grepped the callers of `planGatePass`, `ValidateFormat`, `ValidateFormatAfter` and `ValidateDispatch`, to check where D6–D8 would run.
5. Grepped impactset guards, `commentlint` directives and the testtiming keep files, to see how existing marker parsers treat stacked directive lines.
6. Read testtiming `keep.go` and impactset `onlyKeepLinesAfter`, to confirm the stacking rules.
