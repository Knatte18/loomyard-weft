```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, I list what it forces outside the lines it touches.

**Per change**
1. **Sampler Drift rows** in `sampler-recipe.yaml`.
   - The set I found: `loomshed` row names, `interruptpolicy` (keyed by name, so it already covers the Drift rows), `samplerwiring` stamp, the recipe tests, `loomrecipe/doc.go`, `shedrecipe/doc.go`, and `docs/overview.md`.
   - Covered, except the `docs/overview.md` regression-rows section (F1).
2. **`fixRoute` change** (Sampler-Session route).
   - The set I found: `verify.go`, `driftengine/doc.go:586`, and `verify_test.go` (a sampler case was added).
   - Covered.
   - `amendRows` (`amend.go:33`) and `deferHaltRows` (`verbs.go:35`) list `expectRow`, `darnWriteRow`, `websterRow`, `verifyRow` and `recheckRow`, but not `Sampler-Session`.
   - A halt after `goto Sampler-Session` is a Sampler-Session state, not a Drift stage. The row has no gates, so it cannot Stuck. This is a consequence, not a defect.
3. **`planWriteEngine` swap** in `loomcli/wiring.go`.
   - The only other recipe-shape consumers are `drivewiring.go:133` and `arbiterprompt.go:41`, which already used `planWriteEngine`.
   - Covered. Tapestry still has Plan-Write, so it still gets a plan directory. Darn and sampler do not.
4. **Refusal and help text** in `driftcli` (`cli.go`, `doc.go`, `wiring.go:103` and `:186`).
   - All three refusals now name sampler.
   - The `amend.go:42` help text lists the admitted rows and needs no change.
   - Covered.
5. **`template.yaml` comments** (`enabled`, `scope`).
   - The `scope ""` claim matches `baseline.go:143`, which defaults to full.
   - The comments alone change no config meaning, so no `Reinterpretation` is owed under PATTERN-config-layers.
   - Covered.
6. **PATTERN declared changes**: the diff touches neither `PATTERN.md` nor `pattern/`. The directive declares none, so nothing is owed.
7. **Commit messages**: no `Pattern-Departure:` trailer is present, and none is needed.

**Findings**

- **F1, LOW, consistency**, `docs/overview.md:1224-1230`.
  - The regression-rows list still says "Each carries the entry gate `drift.enabled`".
  - The ordering sentence "`Drift-Expect` comes before `Webster` in loom and before `Darn-Write` in darn" omits the sampler.
  - The diff added a sampler paragraph at about line 1010, but did not update this section, which now underdescribes the sampler rows.
  - Fix: add "and before `Sampler-Session` in sampler" to `Drift-Expect`'s ordering sentence.
  - Fix: add that the sampler's rows also need `drift.scope == full`, or point to the sampler paragraph.
  - Evidence: `docs/overview.md:1224-1230`.

**Tool calls**
1. A grep for stale "tapestry or darn" and "no Drift rows" text. My context had only the diff hunks, not the rest of the repo.
2. A grep for `Darn-Write`, `Webster` and `darnWriteRow` in `driftcli` and `driftengine`, to find row-admission lists such as `amendRows`. The diff did not show them.
3. A grep for the `NameDrift*` consumers, to find other recipe-shape readers. The diff did not show them.
4. A read of `docs/overview.md:1215-1235` and a grep for sampler mentions in `driftengine` and `driftcli`, to check the regression-rows prose. The diff covered only the hunk at about line 1010.
