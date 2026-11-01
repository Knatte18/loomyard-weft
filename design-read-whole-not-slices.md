# Agents read a file whole or all its spans in one message, not in slices

`read-whole-not-slices` · Task · enhancement, webster, stencil · running · PR-Review

## Seen (orch, 2026-11-01)

The five heaviest Webster forks read one file in three to five slices over as many turns, such as `sed -n 200,380p`, `380,560p` and `560,760p` of `internal/loomcli/decision_test.go` in three turns, and re-read the same files in four to seven separate turns.
About 80 of their 208 single-call read turns could have been issued a turn earlier, since the path or line range was already known.

## Idea

The Webster implementer stencil, and the batching sentence every agent gets, ask for a file to be read whole, or for every span the agent needs from it, in one message, and for a file read once to be kept in mind rather than read again.
Note `edit-with-edit-tool` removes part of this, since the Read tool reads whole files or large spans.
