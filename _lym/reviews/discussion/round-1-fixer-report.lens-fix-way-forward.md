```yaml
lens: fix-way-forward
outcome: reported
```

Checked the ways forward the fix added to the decision record: 6 findings (4 MEDIUM, 2 LOW).

1. **MEDIUM — the D5 marker names only one commit.** Changed text: D5, "Before it first detaches, the leader writes a scratch marker naming the chain commit" and "restores a task worktree found clean and detached at the marker's commit".
   - Bisection checks the worktree out at more than one prefix commit, but the marker is written once, before the first detach.
   - Say a leader dies during a later bisection step. Its worktree is then detached at a commit the marker does not name.
   - That state matches neither the restore ("at the marker's commit") nor the Stuck case ("no marker"). No way forward exists for it.
   - Evidence: D7, "each step gates the middle prefix", against D5's marker sentences.
   - Fix: rewrite the marker at each checkout, or restore whenever the marker stands, whatever the HEAD.

2. **MEDIUM — the no-marker Stuck names no way forward.** Changed text: D5, "a detached worktree with no marker is Stuck naming its commit".
   - The restore never acts without a marker, so every resume meets the same Stuck again.
   - The operator's step is to check the task branch out in the worktree, then resume, and the record does not say so.
   - Evidence: D5's text. Every Stuck in this code names a way forward, for example `finalize.go:715` and `:749`.

3. **MEDIUM — D6's "the next Finalize writes the landings" may never come.** Changed text: D6, "A lock taker that is not a run, such as `lym fabric pull` or `lym fabric anchor`, leaves a standing mark alone, and the next Finalize writes the landings."
   - With a standing mark, the parent is ahead of upstream by commits that are not owed tips.
   - `lym fabric pull` reads only an ahead-by-owed-tips prime as level (`internal/fabricengine/pull.go:232`, `:589-593`), so it stops.
   - If no run is queued on that parent, no Finalize ever runs the backstop, and the stop is never cleared.
   - Fix: have the non-run lock taker run the backstop, or name the step that clears it in its stop's way forward.

4. **MEDIUM — D2's rejoin rule contradicts D8's `failed` and `conflict` paths.** Changed text: D2, "keeps the outcome only while its task branch tip is still the entry's task head", against D8's `failed` and `conflict` bullets.
   - A `failed` run's fixer commits, and a `conflict` run's lock-free merge commit, both move the task tip past the entry's task head.
   - If the run's passes after that rewrite its entry as a rejoin, D2 clears the outcome.
   - The run then leads or joins a batch, which contradicts D8's "keeps the outcome … lands alone; it leads no batch".
   - Evidence: D2's rejoin sentence; D8: "`conflict`: … It keeps its outcome, then takes the lock again"; D7's culprit goes to its fixer.
   - Fix: say whether these passes count as a rejoin.

5. **LOW — D8's already-landed path writes a ledger landing whenever one is missing.** Changed text: D8, "A run whose work the parent holds writes its ledger landing if it is missing and takes today's already-landed path, whatever its outcome says."
   - A ledger landing exists only while its push is owed (`internal/pushledger/ledger.go:73`), so it is also missing once the push is done.
   - This now covers every held run, for example a Finalize re-entered after its done mark, so it would write a fresh owed landing for a tip origin already holds.
   - Fix: write it only while the parent's upstream lacks the landing commit.

6. **LOW — D8's `conflict` path can still run a conflict session under the lock.** Changed text: D8, `conflict` bullet, "takes the lock again at the front of the queue and runs today's merge, gate and fixer alone".
   - If the parent moved after the lock-free merge, today's merge under the lock goes through the resolver again (`finalize.go:1404`).
   - The new criterion "its conflict session runs with no parent lock held" then does not hold on that path.

Tool calls:
- grep of `internal/pushledger/ledger.go` declarations and `internal/fabricengine/pull.go` ahead handling: my context lacked the ledger landing's lifetime and how `lym fabric pull` treats a parent ahead of upstream.
