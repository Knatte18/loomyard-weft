# Round 1 fixer report

All eleven findings are fixed in `_lyx/discussion/decision-record.md`, with ledger entries in `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` exits 0 on the result.

## Changed

- **F1** — D2 now runs `go vet` over the set under `integration`, `tmux` and `llm`, and computes the set with `go list -deps -test -tags integration,tmux,llm`.
  D2's bound states the round only compiles the `tmux` and `llm` tiers, and D1's last row says where `llm` is compiled.
  An acceptance criterion covers a tagged-file change.
- **F2** — D4 states what `publish_verify` can run, that a task can edit its own value because `landing.yaml` is per-worktree, and what bounds that (additive; the full plan verify still runs; the edit shows in the PR diff and Finalize's notice).
  The Bound line points at it; Q14 records the pick.
- **F3** — D5 states the bound for a missed marker, defines a scanning test, fixes the run tag (`-tags integration`) and fails derivation for a marker in a `tmux` or `llm` file.
  Two acceptance criteria were added: a misplaced or mis-tagged marker, and every existing scanning guard carrying it.
- **F4** — D7's scope is line-level: a line is checked only when it or its successor is added or changed.
  The finding rule and acceptance criterion follow; Q16 records the pick.
- **F5** — D6 allows a break after a colon, so D6 and D7 agree.
  D7's sentence end admits a closing `)`, quote or backtick.
- **F6** — D2 uses rename detection, maps a renamed or deleted file through its old path, and counts a deleted file in a vanished package under the fifth fallback.
  D7 uses rename detection and passes the round-gate lint when D2 has no base.
  Acceptance criteria were added for rename, deletion and the no-base lint.
- **F7** — An acceptance criterion for the empty set was added.
- **F8** — D3 gets a retention rule and a reworded lookup; an acceptance criterion and Q17 were added.
- **F9** — D2 falls back to the full verify when the derived command exceeds 8000 characters; the fallback list, acceptance criterion and Q18 follow.
- **F10** — The ordering moved into D4 as a sentence; the Open risks bullet now says the ordering is not optional.
- **F11** — Removed D1's tally, D7's incident clause, D9's measured numbers, the first Auto-mode assumption and Open risks four and five.
  The PATTERN-wins clause was extracted into D6 first, and the "not compared with `TESTS`" clause into D9.

## Deferred

None.
