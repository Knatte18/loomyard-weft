# Retro prompt fixes: pin tool facts in decisions, drop the cat clause

Two prompt-text changes from the retro of 2026-11-17 (`docs/research/retro/retro-20261116-115411/report.md`, proposals 2 and 4).

## Discussion-Review flags an unpinned tool fact

`contracts/stencils/loom/loom-rubric-discussion-review.md` gains an `Also flag` bullet.
A Decision or Constraint that rests on how git or another tool behaves must state the observed result and where it is pinned:
a test, a recorded command and its output, or the tool's documentation.
A record that states the behavior without the result or the pin is a finding.
In `decision-record-replace`, reviewers and a fixer in three separate rounds each rebuilt a scratch git repository to learn what `git commit --allow-empty -- <path>` commits when another path is staged,
because the record stated the behavior without the result.

## The edit directive no longer forbids reading by shell

The Claude engine's edit directive (`contracts/stencils/engine/claude/edit-directive.md`) said it overrode any harness text that permits reading with `cat`, `head` or `sed -n`.
Every sampled darn writer, code reviewer and plan writer read its first files with `cat`, batched with other lookups in one Bash call, which is the batching the same prompt asks for.
No guard refused it and it caused no fault.
The operator chose to delete the clause and add no guard row.
The directive still overrides harness text that permits changing files by shell, matching the Codex copy's wording,
and it still tells a role to read with Read and search with grep.
`internal/editdirective/doc.go` drops "reading" from its account of the override to match.

## Design decisions

- The rubric bullet sits beside "A deferred research question", which it narrows: that bullet asks for evidence behind any fact a design rests on, this one names the tool-behavior case and what counts as a pin.
- No test pins either stencil's wording, so none changes.
