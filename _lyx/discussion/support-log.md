# Support log — run-parentage

## Interview

The session ran autonomously, so every turn below is a question the writer put to itself after reading the board and the code, with its pick.

1. Board context.
   `run-parentage` folds in #341 and #338 plus the parent-review single-round hole.
   Neighbours read: `webster-recovery` and `operator-surface-fixes` (sibling bug bundles, no overlap), `pair-lifecycle` (its `batten-resume-sees-stale-blocked` item is the same symptom as #338 on a stale status), `creel` (deferred mailbox, so no queue here), `batten-llm-friction` (unrelated).
2. Where is the parent written and read today?
   Written from `LYX_STRAND_NAME` in batten's arm and loom's bootstrap, copied by Seed-Child; read by `hubgeom` into reed's `LYX_PARENT` and by the parent-review wiring as the reviewer.
3. What identifies "the worktree it is seeded from"?
   `lyx fabric add` forks from the worktree it runs in and already writes a write-once origin record with `parent_branch`.
   Pick: add `parent_worktree` there (Decision 1).
   A hand-run `lyx loom start` seeds from inside the pair, so only the creation point knows the parent worktree.
4. Keep the seed's parent?
   Seed decoding is strict, so the field cannot be dropped from the struct.
   Pick: stop writing it, keep it readable for runs in flight (Decisions 2 and 3).
5. Store the resolved name or resolve at use?
   The orch name is a pure function of hub shortname, parent slug and the orch role.
   Pick: resolve at use, no liveness in the resolver (Decision 3).
6. The board's open question: no orch in the parent worktree.
   `lyx orch` runs only in the prime, so a pair parent never has one, and the prime lacks one briefly during a relaunch.
   Pick: no queue; failed sends stay no-sends; only the parent-review gate holds its prompt until the reviewer is live, inside its existing wait bound (Decision 4).
7. Parent-review rounds.
   The gate opens a request only when none exists and passes once a reject is consumed; the recipe's `attempts: 1, pass_on_cap: true` lets the next arrival through.
   The engine's attempt count is in memory and resets on attach.
   Pick: new round per rewrite, cap of 3 rejects counted from the store, failing the entry at the cap, approve as the way out (Decision 5).
8. What does escalation look like?
   A failed must-pass gate makes Discussion-Write gate-failed Stuck and the run halts `blocked`; the llm driver then messages the parent.
   Pick: that path, no new halt state.
9. Batten's halted arms.
   `paused`, `blocked`, `failed` are hard errors today; `awaiting` is already a budget-exempt wait.
   `lyx loom start` resumes all three halted states.
   Pick: all three become budget-exempt waits (Decision 6).
10. How does batten "tell the parent once"?
    Batten is Go; the only Go route into a Claude session is typing into its pane, which the orch watcher owns behind its idle probe.
    The child's llm driver already messages `LYX_PARENT` at every non-done stop.
    Pick: no batten message; one Warn per halt episode and a stuck reason (Decision 7).

## Rejected alternatives

- Record the parent as the caller's strand name with a prime-orch fallback when the env is empty: keeps two sources, and a caller in a pair still overrides the worktree rule.
- Store the resolved orch name in the seed at seeding time: duplicates a value derivable from stable inputs, and a shortname or role change would leave stale copies.
- Derive the parent worktree from `parent_branch` by finding the worktree with that branch checked out: a git query on every reed launch, and wrong once the parent worktree switches branch.
- Treat a legacy origin record with no `parent_worktree` as parented by the prime: a guess; the legacy seed name is the recorded fact for in-flight runs.
- Queue undelivered upward messages and have the orch watcher deliver them at idle: a mailbox, which is `creel`, deferred by the operator.
- Log-only for parent-review too, with no hold: the gate's three one-minute-throttled prompts burn out during an orch relaunch, and the request then times out unreviewed.
- Walk the worktree parent chain to the nearest orch: no current use creates nested pairs that need it.
- Keep `pass_on_cap: true` and raise `attempts`: the cap would still let the last rewrite through and the in-memory count resets on attach.
- Keep `Final`'s Warn-and-pass on an unconsumed reject: a lost session or a deadline after the cap's reject would let the rejected discussion through.
- Let the recipe's in-memory `attempts` count decide the cap: it fails only on the fourth reject and resets on attach.
- Escalate by halting `awaiting` with `lyx loom approve`/`reject`: those verbs are PR-Gate's, and the review verbs live under `lyx loom review` to avoid that collision.
- A cap of 1 or 2 rejects: too few for a writer to converge on a parent's findings; above 3 spends the parent's attention for little gain.
- Batten typing a notice into the orch pane through reed under its own idle probe: two typers in one pane can collide with a cycle's `/clear`.
- Batten waits through `paused` and `blocked` only: a `failed` child leaves the same watcher gap and is resumed by the same command.
- A time bound on batten's halted wait: the outer run has no better move than waiting, and `lyx batten pause` already stops it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): where is the parent worktree recorded? — fabric's origin record, `parent_worktree`.
- Q2 (resolved, auto-pick): does the seed keep a parent? — no new writes; the field stays decodable for the legacy fallback.
- Q3 (resolved, auto-pick): stored or resolved name? — resolved at use through `agentname`.
- Q4 (resolved, auto-pick): no orch in the parent worktree — no queue; parent-review holds its prompt within its wait bound.
- Q5 (resolved, auto-pick): reject cap — 3, counted from the round store, failing the entry and halting `blocked`.
- Q6 (resolved, auto-pick): way out of a capped halt — `lyx loom review approve` supersedes the latest reject.
- Q7 (resolved, auto-pick): does batten wait through `failed` too? — yes.
- Q8 (resolved, auto-pick): does batten message the parent itself? — no; the child's driver does.
- Q9 (resolved, auto-pick): does the parent-review timeout change? — no, it still passes with a Warn.
- Q10 (open, for the hub): drop `batten-resume-sees-stale-blocked` from `pair-lifecycle` once this lands.
- Q11 (resolved, auto-pick): does an unconsumed reject still pass at a finalize? — no; `Final` fails it and it counts toward the cap.
