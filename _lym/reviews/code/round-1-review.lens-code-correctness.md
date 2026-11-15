```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over `wake`, `sessionStopWanted`, `gateHead` with `gateRetryHint`, and `prContinueVerb`. I ran each case by hand from the inherited context.

**Per function and case**

- `sessionStopWanted`, merged PR or `rec.ReviewEnded`: returns true on any tree, so `wake` re-reads the tree. Correct.
- `sessionStopWanted`, `Continue` with a clean tree at the gated head: returns true, so `wake` stops the session and re-reads. `judge` then ends via `endReview` only on a tree read after the stop. Correct.
- `sessionStopWanted`, `Continue` with a dirty tree or an ungated head: returns false, and `judge` returns "keep waiting" without ending.
  - The stale first read is used only for a verdict that cannot end the review.
  - The session is no longer stopped and resumed at every poll. Correct.
- `sessionStopWanted`, `Continue` with a mailed, ungated head (the deadlock case): returns false, so the session stays put. After `pr-continue` clears `MailedHead`, the next wake has `gateDue` true, forces a poll and gates. If the gate passes in that wake, the record is re-read with `GatedHead == head`, the clean tree stops the session, `readTree` runs again, and `judge` ends. Correct.
- `sessionStopWanted`, open PR approved without `Continue`: unchanged. It is a superset of what `judge` accepts, so a skipped re-read cannot end a review. Correct.
- `sessionStopWanted`, closed PR with neither `Continue` nor `ReviewEnded`: returns false, so `judge` returns Stuck on the unstopped session.
  - The Stuck reason does not read the tree.
  - `Call` stops the session on every verdict, per the doc. Correct.
- `sessionStopWanted`, `Continue` with a merged PR: the merged case wins first. Correct.
- `wake` when the gate runs and the head moves mid-gate (`gateHead` returns false without a write): `head` and `dirty` are stale, but `rec` is unchanged. A moved head differs from `GatedHead`, so no stop and no end. Correct.
- `gateHead` failure path: `mailFailure` receives `reason + gateRetryHint`, and `MailedHead` is written under the lock.
  - It touches only `MailedHead`, so a `Continue` flag written meanwhile survives.
  - A `pr-continue` that runs while a gate is in flight leaves `Continue` set and `MailedHead = head` once the gate fails.
  - The row then waits quietly and the agent has the retry mail, so a second `pr-continue` (`Continue` set, `MailedHead != ""`) clears it. The retry is not lost. Correct.
- `gateHead`, `rec.MailedHead == head` branch: unreachable from `wake`, because `gateDue` already requires `head != MailedHead`. The behaviour is the same as before the diff, so this is not a finding.
- `gateRetryHint`: appended to every gate failure. Its wording conditions the retry on the failure lying outside the change and on the operator having asked to continue. A genuine failure retried with `pr-continue` fails again and mails once more, with no loop. Once per head per `pr-continue` is as designed. Correct.
- `prContinueVerb`, no record: refused, unchanged. Correct.
- `prContinueVerb`, `ReviewEnded`: no write. Correct.
- `prContinueVerb`, `Continue` with empty `MailedHead`: no write, idempotent. Correct.
- `prContinueVerb`, `Continue` with `MailedHead` set: writes, clearing `MailedHead` and keeping `Continue`. Correct.
- `prContinueVerb`, neither flag set: writes both. Correct.
- `prContinueVerb`, `ErrNoRecord` from `UpdateRecord`: the same refusal. Correct.
- `prContinueVerb`, read outside the lock and unconditional write under it: the worst case is a harmless extra write. Correct.

**Findings:** none.

**Tool calls:** none. The manifest context already held `prreview.go`, `prcontinue.go`, the diff and the tests.
