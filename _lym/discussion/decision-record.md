## Goal

```markdown
Imported from [issue #183](https://github.com/Knatte18/loomyard/issues/183).

What happened: `tools/godocreflow` was run over trimmed Go files to clear 624 wrapped-break findings from `lym loom lint-comments`.
Findings rose to 916.
It skips whole comment blocks that hold a heading or list, and it joins lines past the length limit.
The commit was dropped and the findings were fixed by hand.

Why abnormal: the reflow tool is meant to fix these findings and produced more of them.

What to do: make godocreflow process blocks with headings or lists, and make it respect the line length limit, or stop pointing at it as a fix.

Provenance: task doc-go-trim-pass; note Darn-Write.md.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): Accept round 1: all findings fixed

Decision: Accept the code review at round 1. The BLOCKING finding layout-leaves-changed-lines-over-limit is fixed in 509a3cd74: a paragraph whose layout adds a long line is kept as written. F2 to F5 are fixed with tests in the same commits.

Rationale: Every finding has a committed fix and its gates passed; another round would only confirm them.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
