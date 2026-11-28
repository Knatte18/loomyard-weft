```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens on the change set to `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md`: three findings.

- **MEDIUM**: D2's rejoin rule conflicts with D8's `failed` bullet and with D7's culprit handling.
  - Changed text: D2, "A run that joins again at Finalize rewrites its entry with its new task head, and keeps the outcome only while its task branch tip is still the entry's task head."
  - Summary: a culprit marked `failed` moves its task tip as soon as its fixer commits. Read this way, its next pass rewrites the entry and drops `failed`, so the next leader can batch it again. D8 forbids that: "`failed`: the run keeps the outcome, so no leader takes it, … it leads no batch". D7's F5 fix also relies on `failed` lasting until the culprit lands alone.
  - Evidence: D2 leaves open whether a fixer's return through the passes counts as "joins again". If it does, `failed` and `conflict` are lost after any fixer commit. D2 needs an exception for `failed` and `conflict`, kept until the run lands alone, or a statement that a return from the fixer is no rejoin.

- **LOW**: D10's title and its old criterion no longer match the new D10 text.
  - Changed text: D10's body (F2) against its unchanged title and the criterion "D10: a run whose landing goes through a pull request writes no queue entry, and a leader never takes it".
  - Summary: under the new D1 and D10, only a run known before the lock to need GitHub writes no entry. A run the resolver sends through a pull request under the lock may already have an entry and be in a batch before it is left out. So "writes no queue entry" holds only for the run known before the lock, and "never joins a queue" in the title no longer holds as written.
  - Evidence: D10 now says "A member the resolver sends through a pull request is left out of the chain". The new D10 criterion says "a member whose pull-request record appears after it joined is left out of the batch".

- **LOW**: D7 contradicts itself about who writes the landing failure record.
  - Changed text: D7, "only a run's own gate on its own path writes them", beside "A leader that is its own batch's culprit … writes its landing failure record from the prefix-1 gate's log".
  - Summary: the prefix-1 gate is a batch gate run with no failure recorder. A failure record written from its log is therefore a batch-gate record, which the "only … on its own path" sentence rules out. The sentence needs the leader-culprit exception.
  - Evidence: both sentences are in D7 as rewritten for F14 and F16.

No further tool calls made.
