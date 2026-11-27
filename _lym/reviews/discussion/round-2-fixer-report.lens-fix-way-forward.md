```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether each refusal, stop, next step and test the fix added can be reached and leads to progress.

**Ways forward the fix introduced:**
- **(a)** D5: when the parent is owned elsewhere, the pusher stops, posts one notice naming `lym adopt`, and pushes again once this machine adopts the parent back.
- **(b)** D5: when an owner read cannot reach origin, the push stays owed and is retried at D2's interval.
- **(c)** D5: the catch-up reads the commits the hold keeps back as owed to the pair's pusher, and starts the run.
- **(d)** D5: when batten's Land row cannot reach origin, it stops as `git-transport` and D6 waits it out.
- **(e)** D3: a hand `lym fabric push` clears the refused-push record, and the next dobby tick drops the entry or starts the pusher.
- **(f)** D13: while the run's landing entry stands, Worktree-Teardown ends done with the pair standing, and the dobby's landed-pair teardown removes the pair later.
- **(g)** The new D5, D3, D9 and D13 criteria.

Items (b) through (g) reach their states and lead to progress.

**Findings:**

- **MEDIUM: D5's "A parent owned elsewhere stops the pusher with one notice naming `lym adopt` … so the pusher pushes again once this machine adopts the parent back".**
  - Summary: adopting back does not lead to a push in the case that brings this state about.
  - Evidence:
    - While the other machine owns the prime, its landings advance origin's main, so the local main is no longer ahead of origin by owed tips alone. D1 says the prime's code branch is pushed "fast-forward only, never forced", so no push can land.
    - `lym adopt root` runs "the catch-up over the hub, the deploy when root is a subject" (`internal/adoptcli/cli.go:25-50`). That deploy pulls the prime through the same non-fast-forward path as F1 (`internal/fabricengine/pull.go:593-621`). The fix exempts only batten's Land row from that path, so adopt stops there too.
  - The record names no way forward for a local parent that holds owed tips while origin's parent has moved on.

- **MEDIUM: the same sentence, "stops the pusher with one notice".**
  - Summary: no record keeps the pusher stopped or holds the notice to one.
  - Evidence:
    - The landing target stays owed, and D3 has the dobby tick's ahead check start a pusher for an owed target.
    - `runpusher.Start` refuses only on a held lock, a refused-push record or a pause (`internal/runpusher/start.go:118-130`).
  - Without a refused-push record (D7) for the owned-elsewhere stop, each tick restarts the pusher, which reads the owner again and notices again. With such a record, the way forward that clears it has to be named.

Tool calls beyond my context:
- Grepped `internal` for the `lym adopt` command, to check that the named way forward exists.
- Read `internal/adoptcli/cli.go:1-80`, to learn which steps `lym adopt` runs.
