# Decision record — board-push-race

## Goal

A rejected push is called a divergence, and mailed with the merge or reset way forward, only when a fetch after the rejection shows origin holding commits the local branch lacks.
A rejection that outlasts the retry while origin holds nothing the branch lacks is reported for what it is, with git's own refusal line and a plain retry as its way forward.

## Scope

- `internal/fabricengine/pushclassified.go`: the classification of a push whose retry ended in a bare rejection, on every side (code, records, bolt).
- `internal/gitrepo/push.go`: the rejection error carries git's refusal line, and its text no longer claims a divergence.
- The tests that pin a bare rejection as `not_fast_forward`, and a new Bolt row.
- `internal/fabricengine/doc.go` and `internal/gitrepo/doc.go` where they describe the rejection class.
- Out of scope: a new lock around board pushes (D4), and any change to the replay retries themselves.

## Decisions

### D1 — A bare rejection is judged by a fresh fetch before it is recorded

When the retry loop ends with a failure that is a rejection (`gitrepo.ErrPushRejected`) and not a replay conflict or a destructive refusal, the push job fetches the branch's remote and reads whether the upstream holds commits HEAD lacks.
Only a fetch that succeeds and finds such commits records `not_fast_forward`, with the divergence way forward and its mail.
A fetch that succeeds and finds none records class `other`, whose way forward is a plain retry of the side's push step.
A fetch that fails records `transport` when git could not reach origin, and `other` otherwise.
Rationale: the incident's mail followed a rejection whose retry fetch could fail or lose a race with another process of the hub working in the same records repository, and the loop then kept the rejection as the class; the class must rest on what origin holds, not on how the push failed.
Bound: it adds one fetch, only on the failure path after the backoff ran out; it never pushes, merges, rebases or forces, so it can change only a record's class and way forward.
A replay conflict and a destructive refusal stay `not_fast_forward` unjudged, since a replay runs only when origin held commits the branch lacked.

### D2 — A non-divergent rejection is still recorded and mailed once

A rejection that outlasts the backoff with origin behind the branch, such as a declining remote hook, writes an `other` record and sends its one notice, as any unlanded push does.
Its error still wraps `gitrepo.ErrPushRejected`, so `IsPushRejected` and `errors.Is` callers keep matching it.
Rationale: the operator's design drops the mail only where the push goes through; a push that never lands still needs a notice, but not one that names a divergence.

### D3 — The rejection error carries git's refusal line and no divergence claim

`gitrepo.PushRebaseFree` returns `ErrPushRejected` wrapped with the last stderr line that holds a rejection trigger, such as `! [remote rejected] main -> main (pre-receive hook declined)`.
The sentinel's text becomes `gitrepo: push rejected`, dropping `(remote diverged)`.
Rationale: the record's error is quoted in the notice; it must say why git refused and must not assert a divergence D1 did not find.
No caller compares the sentinel by identity, so wrapping it changes no match.

### D4 — The overlap is answered by the judgment, not by a commit-and-push lock

The board's pushers are `lym board sync`, which commits under `board.lock` and pushes under `board.push.lock`, and the callers of `Bolt.PushRecorded` and `Bolt.Push` outside it (owner set, hub config commit and reconcile, orch rules, fabric hub-config paths, the anchor marker), which push with neither lock;
the Bolt retry alone takes `.gitrepo-push.lock`.
The board checkout is the records repository's main worktree, so every weft push and fetch of the hub works in the same repository and its remote-tracking refs.
The incident's logs and creel journal for 2026-10-30 are pruned, so the two processes are not identified by pid; any of these lock-free pushers or a weft push's fetch overlapping a board sync fits the reflog.
No lock is added: holding the board write lock across a network push would make every board write wait on origin, and making every Bolt push take `board.push.lock` would deadlock `Bolt.Sync`, which already holds it around its push.
Rationale: two overlapping fast-forward pushes of one linear history leave nothing to settle; the existing retry lands the loser, and D1 keeps a loser that runs out of retries from naming a divergence.
Bound: overlapping pushes still cost a retry, and in the worst case an `other` notice whose way forward is a plain retry.

## Constraints

- `PATTERN-fabric-git`: git on the board and wefts runs only in `internal/fabricengine`; the judgment fetch goes through the push job's `gitrepo.Repo`.
- `PATTERN-gitrepo-client-boundary`: the fetch and the upstream read use existing `gitrepo` methods, no raw git.
- `PATTERN-push-both-sides`: a push refused after retry still leaves a failed-push record and a creel notice.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: tests stay in the existing tier1 integration file, with injected backoff sleeps and hermetic bares.

## Auto-mode assumptions

- The incident's mechanism is read from the code, since its logs and mail are pruned: a retry that failed at a fetch, or a rejection that outlasted the backoff, recorded as `not_fast_forward` without asking origin.
- A remote hook that declines every push is the test stand-in for a rejection that outlasts the backoff with origin behind.

## Open risks

- A fetch that fails for a non-transport reason during the judgment yields `other`, which loses the divergence call for a branch that did diverge; the next push's rejection, with a working fetch, records it.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - fetch-failure-branch-untested (class consistency, severity MEDIUM, open in rounds 1)
  - missing-upstream-judged-divergence (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: on the code, records and bolt sides, a rejection that outlasts the backoff while origin holds nothing the branch lacks writes an `other` record, not `not_fast_forward`, and its notice names no divergence.
- D1: a code push rejected because origin holds a commit the branch lacks still writes `not_fast_forward` with the merge way forward, and a records or bolt replay conflict still writes `not_fast_forward` with its paths.
- D2: the non-divergent record sends exactly one notice, and its error still satisfies `IsPushRejected`.
- D3: the record's error quotes git's refusal line, and the rejection error text no longer holds `remote diverged`.
- D4: the change description names the board's pushers and the answer on the lock; no lock is added.
- The package docs of `fabricengine` and `gitrepo` describe the judged class.
