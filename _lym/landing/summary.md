# Burler: joint commits for shared-file findings, location names fix site

Closes #159.

A burler fixer in source scope had one commit rule: each finding on its own.
A round whose findings change the same files cannot follow it, and the fixer had to invent a grouping.
Separately, a finding's `location` could name where a defect shows rather than the site the fix changes,
which left the fixer searching unrelated files before it could edit.

## What changes

- `internal/burlerengine/prompt.go` (`fixScopeRules`, source scope): findings whose changes touch a file in common form one group.
  The group is fixed together and committed once, green, with a subject naming every id:
  `<module-or-target>: fix <finding-id>, <finding-id> — <one-line what/why>`.
  A finding that shares no changed file with another keeps its own commit.
  The package doc's FixScope section states the same rule.
- `contracts/stencils/burler/burler-review-format.md`: a finding's `location` names the site the fix changes,
  which may lie apart from where the defect shows.
  A reviewer that did not find that site names where the defect shows and opens the `fix` key with `site not found —`.
- `contracts/stencils/burler/burler-step-fix.md`: a `fix` key opening `site not found —` tells the fixer to find the site before it edits.

## Design decisions

- The grouping unit is a shared file, not a shared hunk: a fixer has no interactive staging to split one file's changes between commits.
- The `site not found —` marker uses a dash, not a colon, so an unquoted `fix` value stays valid YAML.
- The marker is prose for the fixer; `verdict.go` does not validate it, so existing reviews stay valid.
- The issue's side notes on the judge-call timeout mismatch and parent-finding enforcement are not addressed here; they are separate defects the entry's "what to do" does not ask for.

## Merge danger

**Door:** two-way
**Blast radius:** burler
