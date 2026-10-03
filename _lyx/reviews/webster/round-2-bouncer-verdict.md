---
verdict: APPROVED
rationale: "The round-2 review's F1-F3 comment and markdown line-break findings are fixed in a8249b6f8, bb1dcf361 and d97942bcb, a whole-diff grep finds no new semicolon-joined comment line outside the two legacy sites both rounds excluded, go build, go vet -tags integration and go test ./... pass at d97942bcb, and card 17's and card 20's Delete groups still have no callers"
---

## Range

`_lyx/shed/verify-publish-hardening/status.json` names `product.parent: main`, and no `_lyx/loom/rework/` directory exists.
The subject is `git diff $(git merge-base main HEAD)..HEAD`, which is `5805c58fb..d97942bcb`: cards 1–20, round 1's six fixer commits and round 2's three.
The round-2 review used the same range at `44214954f`, before round 2's fixer commits.

## The round-2 findings

- **F1** (`a8249b6f8`): each listed semicolon now ends its line, in `mergestage.go`, `mutation.go`, `cardgate.go`, `verifygatereport.go`, `websterengine/doc.go`, `runlevel.go` and `runlevel_test.go`.
  `pwsh.go`'s appended clause is now its own sentence.
  This closes the ledger's `comment-line-break-convention` entry, whose seven sites are exactly these.
- **F2** (`bb1dcf361`): every block the review listed is rejoined to one sentence per line.
  Every rejoined line was written by the diff;
  in `runlevel_test.go` the merge base already shows the whole paragraph rewritten, so the rejoin touches no legacy line.
- **F3** (`d97942bcb`): each listed markdown line now breaks after the semicolon, keeping list-item continuation indentation.

A grep of every added Go comment line in the full range for `; ` followed by a word finds only `render.go`'s friction-role line and `shedrecipe/entries_simple.go`'s `caller leaves it nil; and ShuttleCfg` line.
Both are token edits inside legacy fixed-column sentences that run on into untouched lines, which both the round-1 verdict and the round-2 review excluded.

`go build ./...`, `go vet -tags integration ./...` and `go test ./...` pass at `d97942bcb`.
Round 2's commits change only comments and markdown, and `mergestagetracked_integration_test.go` keeps its `//go:build integration` line.

## Per-card mechanical check

The plan's only `Delete` groups are cards 17 and 20;
their retired symbols still have no callers, and round 2's comment-only commits cannot add one.
No card carries a `Rename` or `Move` group.

## Considered and not raised

- `contracts/specs/webster-spec.md:9` keeps `…own business; see its package documentation…` on one line.
  The semicolon and both clauses are in the merge base;
  the diff only swapped the dash-bracketed list inside the first clause.
  This is a pre-existing structure on a token-edited line, not a sentence this change wrote.
- A few new comments end with `, so <clause>`, e.g. `mergestage.go`'s `…already refuse, so no other fabric writer…`, `gitrepo/worktree.go`'s `UntrackedFiles` doc and `standalonegeom/webstergeom.go`'s `VerifyDir` line.
  `scribe:prose` names `but`, `and`, `or` and the semicolon as the clause-boundary triggers, and `so` here reads as a result tail as often as a coordinate clause.
  Raising it would widen the line-break sweep into a third round over a judgment call, not enforce a stated rule.
- `webster-template-master.md`'s fixer-fork step carries semicolons inside a backtick-quoted, verbatim fork prompt, which is a literal string and not prose to break.

## Focus departures

The round-2 review carries no `## Focus departures` section, so there is nothing to ratify or reject.
