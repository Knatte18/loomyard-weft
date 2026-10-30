# Card 9 — pr-continue-verb

**Create:**
- `internal/loomcli#loomCLI.prContinueCmd`
- `internal/loomcli#prContinueVerb`
- `internal/loomcli#TestPRContinueVerb`

**Edit:**
- `internal/loomcli#Command`

**Extends:**
- `internal/loomcli/doc.go`

**Uses:**
- `internal/pullrequest#UpdateRecord`
- `internal/pullrequest#ReadRecord`
- `internal/pullrequest#ErrNoRecord`
- `internal/shedrun#PullRequestRecord`
- `internal/shedrun#PullRequestRecordLock`
- `internal/loomcli#loomCLI.commitRecordsCmd`

**Intent:**
The added-after-Discussion decisions: the operator ends a review by asking the PR-Review agent to continue, and the agent sets a flag that the shed reads.
The flag is a loom verb, `lym loom pr-continue`, run by the PR-Review session in its task worktree, annotated `clihelp.AudienceRole`, taking no arguments and no flags; card 10's stencil is the role prompt that names it.
`prContinueCmd` builds it the way `commitRecordsCmd` builds its verb and calls `prContinueVerb` with `shedrun.PullRequestRecord(c.location, shedrun.SelfRunID)` and its lock; `Command` adds it to the `loom` subtree.
`prContinueVerb` sets `Continue` true in the record through `pullrequest.UpdateRecord` and writes a JSON envelope naming the pull request's URL and that the run will close it and land through Finalize; a record already carrying `Continue` or `ClosedForContinue` succeeds the same way and writes nothing new.
A run with no record is refused, an error envelope with exit 1: `loom: pr-continue: this run has no pull request record, so there is no review to end; way forward: wait until the PR-Review row has opened the pull request, then run lym loom pr-continue again`.
An unreadable or unwritable record is an error envelope carrying the error.
Bound: the verb only sets a flag in the run's own record; it can let the run leave PR-Review without a GitHub approval, and what it lets through is bounded by the row, which still gates the head and refuses a dirty tree before it closes the pull request (card 13), and by Finalize's `finalize` gate and trailer, through which the run then lands locally (card 15).
`TestPRContinueVerb` (in `commitrecords_test.go`, which is tier0) drives `prContinueVerb` over a temp record: a record with a pull request gets `Continue` set and the envelope names the URL; a second call succeeds and changes nothing; an absent record is refused with the way forward above and exit 1; a record that does not decode is an error envelope carrying the decode error, with exit 1.
The loomcli doc gains a sentence on the verb.

**ImpactSummary:** One new role-audience loom verb over the run's own record; `Command` only gains a subcommand.

**Commit:** 9: lym loom pr-continue: the PR-Review agent's continue flag
