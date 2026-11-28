# Card 12 — landing-queue-docs

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-landing.md`
- `internal/landingshed/doc.go`

**Extends:**
- `internal/heddleengine/glossary.md`

**Intent:** Name the landing queue in the glossary, the `PATTERN-landing` rule and its background, and landingshed's package doc (D11, D12).

`internal/heddleengine/glossary.md` gains, on the line right after the `landing` line:

`- **landing queue**: the runs at Finalize waiting to land on one parent branch, which the parent lock's holder gates and lands together as one batch. Never: merge queue, merge train.`

`PATTERN.md`'s `PATTERN-landing` line becomes, verbatim:

```markdown
- `PATTERN-landing` — Landing on a local parent branch or deciding on a pull request: only in a run's Finalize under `internal/parentlock`'s lock, alone or in a queue batch; landingshed's one resolver decides it; a pull request's review ends only on the operator's word, whatever the run's autonomy. — [background](pattern/PATTERN-landing.md)
```

`pattern/PATTERN-landing.md`'s opening rule line takes the same rule text, and the file gains a `## Landing queue` section after `## Parent lock`:
who leads (the run that takes the lock with no outcome and whose work the parent does not hold), how a batch is built (its own entry, then live entries with no outcome in join order up to `landing.batch_max`, skipping a paused or moved member, as a chain of landing commits fabric writes with no worktree), how it is gated (once, in the leader's task worktree detached at the chain commit, restored on every way out), how a failure is bisected (prefixes under the same lock, the culprit `failed` and fixed by its own fixer), how a prefix lands (one guarded fast-forward of the parent after each member's own checks as reads), who writes each ledger landing (the leader, one per member with its own landing commit, right after the move under the lock; the backstop at the next lock taker after a leader's death), and that a pull-request landing, a batch of one and a run with outcome `failed` or `conflict` land alone.
`internal/landingshed/doc.go` replaces its Finalize paragraph's lock-free-pass and lost-race sentences with the queue's flow: the lock-free merge with no gate, the entry, the lock's holder leading the batch, the outcomes a woken run acts on, and the backstop.

No test: the wording is read at review.

**ImpactSummary:** Prose only across four documentation files; no code or test reads the changed wording.
