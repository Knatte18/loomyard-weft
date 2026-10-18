# Fixer report — discussion round 1

## Changes

- F1 (fixed as proposed, through the review's fallback): `alignment_pr_approve` is a guard key the local layer refuses (`internal/loomengine/config.go:616`), and a task's `run_settings` are written into that layer (`internal/battenshed/doc.go:65`), so it cannot route PR-Gate to the orch.
  D4 now names the operator as the actor, running the step from the prime after landing and deploy.
  The second open risk says the text survives in the record and git history, and that nothing at landing reminds the operator.
- F2: D4's restart step is dropped.
  The first open risk now says the Linux watcher re-execs at its first restart-safe idle tick.
  On Windows, the operator restarts the orch with `lyx orch stop` and then `lyx orch start --continue`.
- F3: D3 restores the file on a failed write inside the write closure, removing it when it was absent.
  A failed commit is reported as written but uncommitted, and the pull-first board writes skip on the dirty Bolt.
  The criterion is changed to match.
- F4: D1 adds a told `Paths.PrimeRoot`, joined from `location.WorktreePath()`, which `isRequestedNote` reads.
- F5: D3's help text and D5's placement sentence say the verb replaces the whole file, so a draft starts as a copy of the current file.
- F6: D3 writes through `PullThenCommitWritten`, and a skip is refused naming `SkipDetail`.
  The plan-writer note names the pull-first sibling, and a criterion covers the skip.
- F7: decided as Q8: every successful run pushes, unchanged content included, and the envelope reports unpushed commits.
  A criterion covers a retry after a failed push.
  The self-check changed the mechanism, as below.
- F8: D3's bound now says the prime-only refusal checks the working directory and not the caller, and that the moved preamble is what bounds the text.
  The open risk about `orch-any-worktree` stays.
- F9: D3 states that the temporary file sits beside the rules file and can be left untracked.
  The bound now says the verb commits only the rules path.
- F10: D2's moved preamble gains the sentence that a rules file's headings start at `###`.
- The support log gains a Round 1 entry, an updated Q5 and a new Q8.

## Self-check

- fix-consistency, MEDIUM: D5's sentence pointed at a path the filled hub-wide section never names. Fixed as proposed: D5 names `_board/_lyx/orch/rules.md`.
- fix-consistency, LOW: Scope's first bullet left out `PrimeRoot`, and no criterion covered it. Fixed as proposed: both added.
- fix-consistency, LOW: the plan-writer test note named the old failure cases. Fixed as proposed: it names the write-failure restore, the uncommitted report, the pull skip, push failure and no change.
- fix-claims, MEDIUM: `configcli`'s hub writes create no temporary file (`internal/configengine/set.go:81`). Fixed as proposed: the clause is cut.
- fix-proposal, MEDIUM, merged with fix-way-forward LOW (retry after a failed push): with an unpushed non-seed commit and a moved upstream, the pull-first write skips as diverged (`internal/fabricengine/bolt.go:260-285`). Fixed differently: D3 now runs `Push` before the pull-first write, and `Push`'s rebase retry (`internal/gitrepo/push.go:221-223`) carries the commit over a non-conflicting upstream change. A failed first push is refused and nothing is written. The fork's alternative, narrowing the claim to an unmoved upstream, was not made, because it would leave a non-conflicting retry refused as diverged.
- fix-proposal, LOW: nothing tells the operator the D4 step is due. Fixed by disclosure: the open risk says nothing at landing reminds the operator, so the step comes only from this record. There is no write surface for a landing hook.
- fix-way-forward, LOW: a watcher that reaches its tick-error cap exits before any re-exec (`internal/orchengine/doc.go:94`, `:399`). Fixed as proposed: the open risk gives Linux the same operator restart after such an exit.

## Disputed

None.

## Deferred

None.
