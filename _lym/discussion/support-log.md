# Support log — later-transition-push-wait

## Interview

### Turn 1 — exploration

Three explore subagents traced the batten transition push, adopt's reads and the loom row push.
Findings: every status write, skipped rows included, calls the seam, which commits and pushes synchronously through `pairteardown.PushRun`;
the takeover check runs only after a rejected records push;
adopt stops no driver elsewhere, so the original machine learns of a takeover from its next push;
batten's later rows are Land and Worktree-Teardown;
PR-Review and Finalize carry their own guards against a repeated act.

### Turn 2 — first design, superseded

Draft: push synchronously only before rows whose act reaches past the machine (Worktree-Create, PR-Review, Finalize, Worktree-Teardown) and on halts, defer the rest to a detached push.

### Turn 3 — parent requirement (creel #2373 from `ly:orch`)

The operator, through `ly:orch`: everything a run does must reach origin in order and complete; the owner machine never waits for a push;
lym's asynchronous push before `71d6ee1a0` failed by leaving commits unpushed, not by being asynchronous; settle how a takeover reads a run whose last steps are not on origin.
Reply #2376: every transition commits only; a background pusher pushes continuously and in order;
a takeover reads the last transition on origin and the original machine's pusher hands over on its rejected push.

### Turn 4 — parent requirement (creel #2377, #2379 from `ly:orch`)

The operator: redone work after a takeover is acceptable; keep the design simple; guard only acts that harm outside the run when doubled, which already check origin; add no machinery to save redone work.
Reply #2378 asked the orch to record both requirements with `lym loom decision add`; #2379 answered that the command is refused while Discussion-Write owns the record and asked for both to be written as operator decisions.
They are D1 and D2; reply #2380.

### Turn 5 — lens review

Six forks reviewed the first record; every fork returned a report. Judgement per finding:

- walkthrough 1, bound-breaker 6, interactions F4 (batten commits into the prime's shared records, missed by a per-pair pusher and the dobby restart): fixed, D3 keys the pusher per records worktree and D5's dobby tick covers the prime.
- bound-breaker 1 (a commit during the pusher's replay can be lost): fixed, D4 has the transition commit take the weft write lock.
- existing-paths 1, bound-breaker 2, interactions F5 and F6 (an owed push's record would make the act rule hold the run and refuse a resume): fixed, D5 and D8 have owed retries write no failed-push record; the overview criterion dropped.
- interactions F1 (a persistent refusal restarts on every commit and tick): fixed, D5 stops restarts while the refused-push record stands.
- interactions F2, walkthrough 2 (no exclusion for one pusher, lost wakeup on exit): fixed, D3's lock and exit protocol.
- walkthrough 3, bound-breaker 3 (the "local pause gate" is the network gate): fixed, D1 drops the gate and D7 names the pause record `lym loom pause` writes.
- bound-breaker 4 (D7's one-row bound fails with origin unreachable): fixed, the bound now states it.
- existing-paths 2, bound-breaker 5, interactions F7 (teardown's archive push only logs a refusal, so unpushed commits could be lost): fixed, D5 has Worktree-Teardown push to completion before removal, and the backoff holds no lock.
- walkthrough 4 and 5, bound-breaker 7 and 9, interactions F8 (hand-over ends the pusher itself; two pushers per slug; a loom hand-over leaves the pair): fixed, D3 registers pushers per worktree, not per slug, and D6 runs them outside the pair and defers to `HandOver`'s existing effect.
- interactions F3 (the takeover window lasts through a long row): rejected, the operator's D2 accepts redone and concurrent local work and guards only outside acts; D6's bound states the window.
- leanness MEDIUM open risk on Finalize's owner check: fixed, the open risk dropped since the operator's D2 rests on those acts' own origin checks, named in D6.
- bound-breaker 8, existing-paths 4 (code-tip window): fixed, D6's bound names the existing code-tip settle.
- existing-paths 3 (two retry loops): rejected, which loop owns a moved tip is plan detail; D3 states the behaviour.
- walkthrough 6 (board writes do not wait): fixed, the Scope's Out line states it with its source.
- board-scope 1 (the operator's requirements only in creel mail): fixed, the orch asked for them in the record as D1 and D2.
- leanness 1 (unchanged `PATTERN-owner-machine` text): fixed, the overview names the pusher's reconcile.
- leanness 2, 3, 4, 5 (duplicated facts, history in Goal and D3, redundant assumptions): fixed, cut.
- leanness 6 (timing as a Review criterion): fixed, moved to the plan notes as a by-hand check.
- leanness 7 (D2 rationale overlap): rejected, the reason is kept.
- leanness glossary note: fixed, D3 gives the pusher's glossary line.

## Rejected alternatives

- Synchronous push before the rows whose act reaches past the machine, deferred elsewhere: rejected by the operator's decision that no step waits for a push.
- Batching several transitions into one synchronous push: still a wait at the batch's end.
- One detached push per transition, as before `71d6ee1a0`: a failed or never-run push left its commits behind; one pusher per worktree with restarts replaces it.
- A pusher per run keyed by slug: batten commits into the prime's shared records and a slug has two seams, so the key is the worktree.
- A takeover-seen mark for the driver to act on: the pusher calls the existing hand-over path itself, and the operator wants no machinery to save redone work.

## Review rounds

_No rounds yet._

## Question ledger

- Which transitions need a push before the act? Resolved by the operator's D1: none.
- How does a takeover read unpushed steps? Resolved by the operator's D2 and D6: from origin's last transition, the rest archived on hand-over.
- Does an owed push still hold the run? Auto-pick: no, D8, since holding is a wait.
- Does Worktree-Teardown wait for the push? Auto-pick: yes, D5, since it removes the commits and nothing follows it.
- Where does the loom pre-push reconcile go? Auto-pick: into the pusher, D7, so it still guards every push.
- Does a refused push retry on every commit? Auto-pick: no, D5, it waits for the refused-push record's way forward.
