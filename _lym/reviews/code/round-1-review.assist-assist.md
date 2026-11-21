---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/burler/burler-review-format.md:59
    summary: The fix-site location requirement conflicts with carried-finding path restrictions
    fix: Qualify the new location rule so a carried finding retains its permitted location when the round rules prohibit moving it, while naming the discovered fix site explicitly in summary and fix. Reserve site not found for sites that remain unknown.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: handler
    location: internal/burlerengine/prompt.go#fixScopeRules
    summary: The bug-fix commit range has no required root-cause trailer
    fix: "Include a Root-Cause: trailer in a commit of this review range naming the defects removed here: the unconditional per-finding commit instruction lacked a shared-file exception, and the finding contract did not distinguish a fix site from a symptom site."
---

### [MEDIUM] Fix-site locations conflict with carried-finding restrictions

**Location:** `contracts/stencils/burler/burler-review-format.md:59`

**Issue:** The new rule requires the actual fix site once the reviewer knows it.
The carried-finding rule in the same stencil permits a moved path only when the changes file lists it.
`contracts/stencils/burler/burler-step-review.md:49` repeats that restriction,
and `internal/burlerengine/roundcheck.go:260` enforces it.
A deferred finding can name `display.go` while its site remains unknown.
If the next reviewer discovers the fix belongs in untouched `settings.go`, the two instructions cannot both be met.
The existing fallback puts the discovered site in `summary` and `fix` while retaining the permitted location.
The new rule needs to acknowledge that fallback without labeling a known site as unknown.

### [MEDIUM] Required root-cause trailer is absent

**Location:** `internal/burlerengine/prompt.go#fixScopeRules`

**Issue:** The decision record alone describes defects in the shared-file commit rule and finding-site instructions.
The rubric requires a `Root-Cause:` trailer for this kind of bug fix.
`git log --format=full 86145ecf7bd69eef6b1342cec056181deceda2f6..HEAD` shows no such trailer.
The explanatory bodies of `a938c6da` and `ada83edf` describe the changes but do not supply the required trailer.

## Other focus checks

- Shared-file relationships join a bridging finding and both neighbors into one group.
  An unrelated finding retains its own commit, and group subjects name every finding ID.
- The package documentation describes that same grouping and subject rule.
  No searched consumer requires a single finding ID in a commit subject.
- The marker agrees between the review-format and fix-step stencils.
  It qualifies the preceding location instruction and leaves the fixer's dispositions intact.
  The source-scope wording permits the finding's actual fix site within the working tree.
- The changed text is static prompt prose; scope-selection behavior and its existing coverage are unchanged.
  No new literal prose assertions are warranted under the test-economy rule.
- Both requested changes are present, with their associated documentation updates.
  The commit messages and landing summary make no quantitative improvement or reference-agreement claim.
  The diff changes no machine-readable record shape, config meaning, data deletion, or outward push behavior.

## Validation

Reviewed the supplied diff pieces, range commit messages, board entry, decision record, and affected readers.
No target files were changed.
