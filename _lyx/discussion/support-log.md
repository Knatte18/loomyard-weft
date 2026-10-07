# run-pushes — support log

## Interview

Autonomous mode: no operator answered; every pick below is the agent's own, grounded in the board entry and the code.

1. Read board entry `run-pushes` (#162): push the task branch wherever Go pushes the records side; a status push survives a diverged remote (#418).
   No other board entry overlaps; `run-recovery` touches crash recovery, not pushes.
2. Explored the push paths.
   Loom's and batten's status seams push the records side only through the lock-free `PushAnchored`.
   `Fabric.Commit` (and so `lyx fabric commit` and webster's `fabricSync`) spawns a detached child that pushes both sides under `fabric.push.lock`, but only when that call landed something, which is why the code branch was pushed by Webster and then fell behind during Burler rounds.
   `lyx fabric push` pushes the records side only; `lyx fabric sync` spawns a records-only child.
3. #418: a lock-free status push concurrent with a locked detached push child on the same records branch can lose the remote ref update, which `PushRebaseFree` classifies as `ErrPushRejected` ("remote diverged"); the next transition then pushing fine fits a race, not a divergence.
   Picked: share the lock, and retry once after a fetch shows the remote tip is contained locally.
4. Scope boundary: batten runs in the prime, whose code branch is the parent; pushing it each transition would publish the operator's local parent commits, so batten stays records-only.
5. Picked to make `fabric push`/`sync` push both sides for agreement with `fabric commit`, and because `commit-records` names `lyx fabric push` as its way forward.
6. Picked a PATTERN entry, since "pushing never rests on agents" is the operator's cross-cutting direction.

## Rejected alternatives

- **Every agent commits through a `lyx fabric` verb that pushes both sides**: the operator already judged agents will keep running `git commit` whatever their prompt says.
- **Tell Burler and other agents to push**: same reason, and the board puts the push in Go.
- **Rebase on rejection (`PushCoalesced`'s `pull --rebase`)**: rewrites SHAs under a running pair and invalidates the correspondence index; a real divergence is the operator's decision.
- **Retry without a fetch, or several retries with backoff**: a blind retry cannot tell a race from a divergence; one checked retry is enough once the lock removes the in-hub race.
- **Reuse `CoalescePushBothAt` as is for the seam**: it swallows a rejection as a warning and skips the records-side push on a code-side error, so `commit-records` could report success with nothing pushed.
- **Push the code side in batten's seam too**: would push the prime's parent branch automatically.
- **Push the code branch inside webster per batch or after each Burler fix**: duplicates the transition seam; the lag of at most one step is what the board asks for.
- **A status push that blocks on the lock with no timeout**: a detached child that stalls on the network would hold the lock and stall every later transition behind it, against the seam's rule that a push failure warns and the run continues.
  The bounded wait removes only that waiting; `gitexec` sets no command timeout, so a status push that holds the lock and stalls in git still stalls its own transition, as a lock-free push does today.
- **Keep `lyx fabric push` on `PushCoalesced`, or route it through `CoalescePushBothAt`**: the first rebases and rewrites SHAs; the second swallows a rejection as a warning, so the verb that `commit-records` names as its way forward would report success with nothing pushed.
- **Keep the prime's code branch out of `lyx fabric push` and `sync`**: the prime is not distinguishable in the layout today, and `lyx fabric commit` already publishes those commits whenever it lands something.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): Which pushes carry the code branch? — Loom's transition seam, `commit-records`, `lyx fabric push` and `sync`; batten excluded. Rationale: Interview 4–5.
- Q2 (auto-pick, resolved): How does a status push survive #418? — Shared absorbing lock plus one fetch-checked retry; real divergence still warns. Rationale: Interview 3.
- Q3 (auto-pick, resolved): Rebase-free or rebasing? — Rebase-free on both sides. Rationale: SHAs and correspondence index stay stable.
- Q4 (auto-pick, resolved): Does a code-side failure block the records push? — No; both are attempted and both failures reported.
- Q5 (auto-pick, resolved): New PATTERN entry? — Yes, `PATTERN-push-both-sides`, review-enforced.
- Q6 (open, not blocking): The #418 drive report was not inspected; the race hypothesis is inferred from the code. The retry covers other non-divergent rejections either way.
- Q7 (auto-pick, resolved): Does a status push wait on the lock without limit? — No; the status side waits a bounded, injectable time and warns on expiry, the detached child and `lyx fabric push` keep the blocking acquire. Rationale: the seam's push-warns disposition; the bound covers a waiter only, since `gitexec` sets no command timeout.
- Q8 (auto-pick, resolved): What does a failure of the new entry return? — Every failed side's error is wrapped with `%w` and names the side and its worktree path; both failed return `errors.Join` of the two, so `errors.Is` still finds a rejection. Rationale: the caller and the `commit-records` envelope must tell which side failed in the one-side case too; only `PushAnchored`'s records-only contract keeps the bare sentinel.
- Q11 (auto-pick, resolved): Does `lyx fabric push` wait on the lock with a bound? — No, unbounded: it is an operator command off the transition path, and after a landing commit it contends with the detached child `Fabric.Commit` just spawned. Rationale: `PushCoalesced`'s lock it replaces also waited without limit.
- Q12 (auto-pick, resolved): What way forward does `commit-records` name after a failed push? — Two rows: a staying rejection names merging the remote branch in the worktree the error names, then re-running; any other failure stays transient. Rationale: under Decision 5 `lyx fabric push` and a re-run both fail again on a real divergence, as the Publish push-rejected rows already treat it.
- Q9 (auto-pick, resolved): Which path does `lyx fabric push` take, and what happens to its rebase recovery? — The new entry; the `pull --rebase` on rejection is dropped, since Q3 keeps SHAs stable. Rationale: the Rejected alternatives entry on `PushCoalesced` and `CoalescePushBothAt`.
- Q10 (auto-pick, resolved): Is the prime's parent branch exposed through `push` and `sync`? — Yes, unconditionally, bounded by a plain rebase-free push; accepted as an extension of what `lyx fabric commit` does. Rationale: the Rejected alternatives entry on keeping the prime's code branch out.
- Q13 (auto-pick, resolved): What does `lyx fabric push` do in the prime when the parent branch is behind, diverged or protected? — Push the records side, then return the error envelope naming the code side; `sync` stays unaffected. Rationale: the prime is not distinguishable in the layout, and Decision 5 makes a staying rejection an error so that the way forward `commit-records` names carries the task branch.
