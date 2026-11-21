# burler: commit shared-file findings jointly; review names the fix site

A burler fix round on the source scope now has a sanctioned way to commit findings that change the same files, and a review's `location` now names the site the fix changes.
Before, the fix prompt demanded one commit per finding, so a fixer facing findings that overlapped on files improvised its own grouping.
Before, `location` named where a defect showed, so a fixer could spend a round reading unrelated files to find the real site.

Closes #159

## What changes

- **Joint commits** (`internal/burlerengine/prompt.go`, `doc.go`):
  the `FixScopeSource` rules have the fixer form groups before it starts fixing.
  Findings whose changes touch a file in common are fixed together and committed as one commit,
  with the subject `<module-or-target>: fix <id>, <id> — <what/why>`.
  A finding that shares no file keeps its own commit and the old subject format.
  The overlay scope is unchanged: it runs no git.
- **Review `location`** (`burler-review-format.md`, `Finding` in `verdict.go`):
  `location` names the site the fix changes, which may lie apart from where the defect shows.
  When the reviewer did not find that site, `location` names where the defect shows and `fix` opens with `site not found —`.
  A carried finding keeps its location path and names a fix site outside it in `summary` and `fix`.
- **Fix step** (`burler-step-fix.md`):
  a `fix` key opening with `site not found —` tells the fixer to find the site before editing.
  The never-push paragraph no longer calls the source rules commit-per-fix.

## Design decisions

- Groups chain: a finding touching a file any group member changes joins that whole group, so every finding lands in exactly one commit.
- A finding whose change reaches a file an earlier commit already changed is committed on its own, never folded into that commit.
  This keeps a landed commit from being amended and keeps `git log` an accurate record of what landed if the round dies.
- The `site not found —` prefix is a convention in the prompt text only; no parser reads it.
- `TestComposePrompt_FixScope` drops its `requireNotContains(…, "commit each fix")` on the overlay output,
  since the source rules no longer use that phrase.

## To check by hand

- The group-formation rule in `fixScopeRules` is prose a fixer follows, with no test pinning it.
  A review left one finding open: joint-commit groups do not chain (class design, severity MEDIUM).
  Check that the chaining sentence in `fixScopeRules` and `doc.go` reads unambiguously.
- No fix round was run against the new prompt, so the joint-commit behavior is unobserved.

## Merge danger

**Door:** two-way
**Blast radius:** burler
