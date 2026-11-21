```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change, I listed what reads, asserts or describes it and checked whether the diff left any of it wrong.

**Per change, the affected set and whether it is covered**

1. `fixScopeRules`, source scope (`internal/burlerengine/prompt.go:322-335`): the per-finding sentence is reworded and a joint-commit rule is added.
   - Caller `composeFixPrompt` (`prompt.go:207`) fills `fix_scope_rules`. Covered: same key, still non-empty.
   - Caller `fix_test.go:206` passes the output through. Covered.
   - `prompt_test.go` `TestComposePrompt_FixScope`, source subtest (`:334-344`): asserts only `"commit"`. Still passes; it pins none of the new rule. The tests lens owns this.
   - `prompt_test.go` overlay subtest (`:355`): `requireNotContains(t, got, "commit each fix")`. **Left wrong**: no scope renders that phrase any more, so the check is vacuous (F1).
   - `prompt_test.go:330`, the doc comment saying "commit-per-fix phrasing": now stale (F2).
   - `burler-step-fix.md:98`, "If your fix-scope rules above name commit-per-fix on the repo": stale wording (F2).
   - Package doc `doc.go:377-382`: updated in the same diff. Covered.
   - `docs/skills.md:188` mentions commit-per-fix, but for webster's fork template, not burler. Not affected.
   - Other fixer stencils (webster, weaver, gatefix and landing fixers) have their own commit rules and are not burler. Not affected.
   - Anything that parses commit subjects for a single finding id: none found. No regexp or `git log --grep` in `burlerengine`, `shedadapters` or `loomcli` reads `fix <id>`. Covered.
2. `location`/`fix` contract in `burler-review-format.md:58-62` (site of the fix, `site not found —` marker).
   - `verdict.go:66`, the `Finding` doc: "Location points at the offending content". **Left wrong**: under the new rule it is the site the fix changes (F3).
   - `burler-step-fix.md:26-28`: updated in the same diff. Covered.
   - `burler-template-fixer.md:29`, "location and fix keys say where to look and what to change": consistent.
   - `burler-step-review.md:51-54` (carry rules; a regression located at the changed line that causes it): consistent with the fix-site rule.
   - `burler-parent-consolidation.md:20`, "give it the location ... the change points at": consistent.
   - `verdict.go`'s `Fix` field: not validated, so the marker parses. Covered.
   - Stencils reaching the seeded `_lym/stencils` copy go through `stencilstore` reconcile (`reconcile.go`). This is the standard path, nothing extra is needed.
3. Package doc `FixScopeSource` bullet (`doc.go:378-381`): matches the prompt text, including the group subject format. Covered. Under `PATTERN-code-conventions`, the bullet documents the package's own prompt behaviour, not a rule other code must follow. No finding.
4. Commit messages over the range: no `Pattern-Departure:` trailer, and no claims of agreement or improvement. The constraints section declares no PATTERN changes, so the "landed" check has nothing to check. Whether a `Root-Cause:` trailer is required falls under the rubric's root-cause bullet, which is outside this lens; I leave it to the handler.

**Findings**

- **F1.** MEDIUM, design. Location: `internal/burlerengine/prompt_test.go:355`.
  - Summary: the overlay subtest's `requireNotContains(t, got, "commit each fix")` guards a phrase the diff removed from the source-scope text, so it can no longer catch source commit rules leaking into the overlay prompt, which its doc comment says it checks.
  - Fix: assert the absence of a phrase source scope still renders, such as "Commit message format" or "joint", or of the joint-commit sentence.
  - Evidence: `prompt.go:328-335` (the new text has no "commit each fix"); `prompt_test.go:330-331,355`.
- **F2.** LOW, consistency. Location: `contracts/stencils/burler/burler-step-fix.md:98`. The same stale wording is also at `internal/burlerengine/prompt_test.go:330`.
  - Summary: both still describe the source-scope rule as "commit-per-fix", but the rules now also commit a group of findings as one joint commit.
  - Fix: reword to something like "name commits on the repo" (stencil) and "the commit-rule phrasing" (test comment).
  - Evidence: `burler-step-fix.md:98`, `prompt_test.go:330`, `prompt.go:328-335`.
- **F3.** LOW, consistency. Location: `internal/burlerengine/verdict.go:66` (`Finding`).
  - Summary: the doc comment says "Location points at the offending content", while the review format now says `location` names the site the fix changes, which may lie apart from where the defect shows.
  - Fix: reword to "Location names the site the fix changes, or where the defect shows when the fix key opens `site not found —`".
  - Evidence: `verdict.go:66`, `burler-review-format.md:58-62`.

**Tool calls**

1. grep across the repo for other statements of the per-finding/commit-per-fix rule. Context lacked any search beyond `burlerengine` for these phrasings.
2. grep for commit-subject parsers in burler, shed and loom, plus other "offending content / where the defect" descriptions of `location`. Context lacked any search for a parser of commit subjects.
3. grep of all stencils for fixer commit rules, plus a listing of the burler stencils. Context lacked the other stencils' commit wording.
4. grep of the burler fix-check, self-check, review, consolidation and solo stencils for commit, location and site. Context had not read them.
5. grep of `doc.go` and `docs/` for location and fixer-commit claims. Context held only the diff's span of `doc.go`.
6. grep of `stencilstore` and `stencils.go` for reseeding. Needed to confirm that the changed stencils reach the seeded copy.
