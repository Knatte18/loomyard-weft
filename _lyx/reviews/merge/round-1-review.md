---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: pattern/PATTERN-verified-tree.md:10
    summary: the background file says the round gate's only site is Webster's verify-gate fixer fork, but the parent's cold fixer is a second site that runs the same fix_gate call
    fix: In pattern/PATTERN-verified-tree.md line 10, change "The `round` gate's only site is Webster's verify-gate fixer fork" to name both verify-gate fixers, the fixer fork and the cold fixer `lyx webster fix-verify` spawns, as the round gate's sites, each running the `round` call through `Verify` with `BaseCommand` set to the told base command. Make the same site-list correction in internal/websterengine/doc.go if it says the fork alone runs the round gate.
    origin: lens:code-pattern-fit
---

### [MEDIUM] PATTERN-verified-tree names only the fixer fork as the round gate's site

**Location:** pattern/PATTERN-verified-tree.md:10
**Issue:** The task rewrote this background file to say the `round` gate's only site is Webster's verify-gate fixer fork, as decision record D8 and its notes for the plan writer prescribe.
The parent then added the cold verify-gate fixer.
Both fixers share `webster-body-verify-fix`, whose required `fix_gate` marker is filled from the same rendered gate call (`internal/websterengine/render.go:336`, `RenderVerifyFixColdPrompt` called at `internal/websterengine/verifyfixer.go:126`).
So the cold fixer is a second site of the `round` call, and the merged background file states a site list the merged code no longer has.
Neither side alone is wrong: the task's list was right before the merge, and the parent never edited the pattern file.

## Rejected

- Retired names left in `internal/loomcli/doc.go` (lines 66, 123, 175, 265), `internal/webstercli/verify.go` (lines 1, 30) and `contracts/specs/loom-plan-spec.md:408`: the parent never touched these lines, and the task's own cleanup left them, so they are not an interaction. The Webster-Review carry-over `loomcli-doc-retired-surface` already holds the loomcli part.
- `pattern/PATTERN-verified-tree.md:3` still naming the Webster-Burler-Fix round gate: a retired segment the task's own edit left behind, not the interaction; the parent's cold fixer does not touch that sentence.
- `docs/overview.md:557` naming `Webster-Bouncer`: it is the restart note for an old in-flight run, a deliberate historical mention.
- `checklist_test.go` fixtures using `Publish` and `PR-Rework` as row names: generic renderer fixtures, not claims about the recipe.
- Missing test for `fix-verify`'s refusal on a gate config with no `round` list: a gap in the parent's own tests, and the envkit package the lens cited does not exist at that path.
- `decisionVerb` doc and help text against the task's edits: different lines, no semantic conflict; the replacement behavior is pinned in `internal/discussionparser/append_test.go`.
- No marker the task removed or renamed is still expected by the parent's stencils or `markers_test.go`; no test became redundant with another.
