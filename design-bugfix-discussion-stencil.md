# Bugfix section in the Discussion-Write stencil

`bugfix-discussion-stencil` · Task · enhancement, stencil, loom · starting · Seed-IDE

From F17's Kiro lead (bugfix spec with a fail-on-base test).
The Go gate half has landed: `lym gate red-first` on the `Darn-Write` row, and the `Root-Cause:` trailer.

## Idea

When Discussion-Write runs for an entry labelled `bug`, its stencil asks for and records:

- the observed fault and how to reproduce it;
- the root cause, or what is still unknown about it;
- the test that will fail on the merge base and pass after the fix, or why no test seam exists.

The decision record then states up front what the red-first gate later checks.
