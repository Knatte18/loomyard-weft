```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. Nothing the diff touches is left wrong by it, so I have no findings.

**Per change, the set I found:**
- **`routedDecisionsLine` (`internal/burlerengine/prompt.go:240`):**
  - The only code reference is `fixWork`, at `prompt.go:231` and `prompt.go:235`.
  - Tests only reference the constant by name, as `wantWork` and a `Contains` check in `fix_test.go:282,283,365`, so the reworded text breaks none of them.
  - No other code, doc or stencil repeats the old "neither fix, dispute nor defer them… leaves them out" wording, which a repo-wide grep confirms.
  - The work statement renders under "What to fix" in `burler-step-fix.md`, and the parent ruling rule sits further down the same stencil, so "the parent ruling rule below" is reachable.
  - Covered.
- **New `Rulings` fixer-report section:**
  - No code parses the burler fixer-report's sections. The only fixer-report parser the sole-parsers entry lists is `landing-fixer-report-spec.md`, which belongs to the landing fixer.
  - `repairReportBeforeGate` (`engine.go:106`) only appends a rewrite instruction and does not check sections.
  - The engine-written skipped reports (`fix.go:33,43`) carry only a `Disputed` heading. No fixer runs in that case, so no ruling can arrive and `Rulings` is not needed.
  - Covered.
- **The no-message rule (`burler-template-fixer.md:43`):**
  - The only other stencils with a "send no message" line belong to the reviewers and the assistant seats, which are different roles.
  - The fixer's wording and the fix step's rule agree, and neither conflicts with another fixer-role stencil.
  - Covered.
- **The `lym creel post --parent` form:**
  - `creelcli/cli.go:413-423` refuses `--parent` only with `--reply` or an extra receiver argument. `--parent` with no `--expect-reply` is a valid form, matching the stencil.
  - The parent directive (`parent-directive-parent.md:11,23`) names both kinds of message, parent mail and the `Parent decision recorded…` notice.
  - Neither `shellguard` nor the engine code I searched denies `creel post` for the fixer, so `PATTERN-refusal-way-forward`'s "no stencil names a step agents' settings deny" holds.
  - Covered.
- **`doc.go` prose:** it matches the stencil behaviour. The older Disputed paragraph at `doc.go:254` concerns dispositions, which this change leaves alone. Covered.
- **PATTERN and departures:** the diff declares no PATTERN changes, and the commit messages carry no `Pattern-Departure:` trailer. Nothing to land or check.

**Tool calls:**
1. `grep` for the old routed-line wording and for "Disputed" across `.go` and `.md` files, plus creel/deny greps in `internal`. I needed to find other copies of the wording and any consumer of fixer-report sections.
2. `grep` for "no message" wording, fixer-report references in `pattern/`, and "Rulings" in `contracts/specs` and `docs`. I needed sibling stencils and spec coverage.
