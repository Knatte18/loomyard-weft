```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward this round's fix introduced (the dobby's exit-record read, the spent-budget listing, the slot taking, and the new D6 criterion) for reachability and progress.

no findings

- D6's exit-record read works. The dobby started the child, so it knows that child's process ID and can match the record.
  - A killed refresh leaves an older record that names a different process ID, so it counts as none left.
  - A restarted dobby that did not start the live child starts no refresh while the `test-map` record exists. After that it has no record of the prime HEAD, so its next tick starts a refresh, and that is progress.
  - A manual `lym gate testmap` call can overwrite the record. That also counts as none left, and the leftover packages wait for the next prime HEAD move, which the changed D6 text states.
- The tests D7 step 4 leaves unrun when the budget is spent are listed as not judged. The skip rule's "a test past the budget never skips the step" covers them, so a later call retries them.
- D7 step 4 takes its slot as a non-tier holder. `internal/gateslot/doc.go:62` defines that class: one slot, its own Go flags through HolderEnv. `PATTERN-gate-slot` uses the same wording, so a plan can carry this out.
- The new D6 acceptance bullet can be tested: a fake child plus an exit record with a matching or a different process ID reaches each branch.

Calls made:
- I grepped `internal/gateslot/*.go` for "holder" to check that the non-tier holder that step 4 cites exists. My context did not have gateslot's doc.
- I read `internal/gateslot/doc.go` lines 36–75 to confirm the contract of a non-tier holder. The grep showed only the doc's line headers.
