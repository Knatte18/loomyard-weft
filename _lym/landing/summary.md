# Pin tool facts in decisions; drop the edit directive's cat clause

Two prompt-text changes from the 2026-11-17 retro (`docs/research/retro/retro-20261116-115411/report.md`).

## Discussion-review rubric: unpinned tool facts

`contracts/stencils/loom/loom-rubric-discussion-review.md` gains a finding: a Decision or Constraint that rests on how git or another tool behaves must state the observed result and where it is pinned (a test, a recorded command and its output, or the tool's documentation).
Without that, each later round re-ran the same experiment; the retro saw three sessions each build a scratch repository to learn what `git commit --allow-empty -- <path>` commits when another path is staged.

## Edit directive: clause on cat, head and `sed -n` removed

The retro found that darn writers, code reviewers and plan writers read their first files with `cat`, batched with other lookups in one Bash call.
None was refused and none caused a fault, and the calls are the batching the same prompt asks for.
The operator chose to delete the clause rather than add a shell guard row that refuses `cat <path>`.

The override sentence in `contracts/stencils/engine/claude/edit-directive.md` now covers only changing files with `sed`, heredocs or short scripts.
The `internal/editdirective/doc.go` package comment is updated to match.
The shell guard is unchanged: `sed -i`, `perl -i` and redirects into source files stay refused.

## Design decisions

- No guard row for `cat`: reads by `cat` are allowed in practice, and a refusal would turn one batched call into several parallel Reads at the same turn count.
- The rubric names the acceptable pins as a closed list of three, so a reviewer can tell a pinned fact from an asserted one.

## To check

- Whether the `internal/editdirective` tests that compare against the directive text still pass; this was not run as part of the description.
