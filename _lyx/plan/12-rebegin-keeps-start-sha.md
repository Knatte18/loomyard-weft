# Card 12 — rebegin-keeps-start-sha

**Edit:**
- `beginbatch.go`
- `beginbatch_test.go`

**Intent:** Keep a batch's recorded `StartSHA` across a re-begin, so batch 1's `StartSHA` stays the HEAD webster recorded before the first batch began — the baseline D7 names and card 9 reads.

Today `BeginBatch` records `StartSHA: head` unconditionally, including when it re-begins a batch whose earlier fork already landed commits (the re-begin path `begunCards` exists for).
The recorded start then names a commit inside the batch's own work, which breaks the promise in `BatchState.StartSHA`'s doc comment and would let triage excuse a regression the first fork introduced as pre-existing.

When `deps.State.Batches[number]` already holds a record with a non-empty `StartSHA`, `BeginBatch` writes that value into the new record instead of the captured head, the same inheritance `recover-batch` applies to a recovery record.
A first begin, or a prior record with an empty `StartSHA`, records the captured head as today.
`BeginResult.StartSHA` carries the recorded value, and its doc comment says so.

In the `integration`-tagged `beginbatch_test.go`, add cases asserting:
a re-begin over a non-terminal fork record carrying a `StartSHA` keeps that value in both the record and the result, with the worktree's head moved past it;
a re-begin over a prior record with an empty `StartSHA` records the current head;
and the existing first-begin case still records the head.

**ImpactSummary:** Changes only the re-begin branch of `BeginBatch`; its readers (`record-batch`'s post-batch delta, `recover-batch`'s inheritance, the `start_sha` field in `webstercli`'s begin-batch envelope, and card 9's baseline) now see the whole bracket's base commit, which `recover-batch` already guarantees for a recovery record.

**Commit:** 12: rebegin-keeps-start-sha
