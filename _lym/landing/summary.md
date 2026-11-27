# Bring back the PR-Review agent session under beam.pull_request review

Under `beam.pull_request: review` the PR-Review row opened the pull request and waited with no agent in the task's window.
Nothing on screen showed the wait, no agent took the operator's feedback,
and `lym loom pr-continue` refused every agent session.
The row now keeps one interactive session in that window again, as it did before the `pr-reviewer` role was retired.
The operator works the pull request with its agent, which answers comments, commits fixes
and ends the review on the operator's word.
`trail` is unchanged.
An `off` landing with no pull request is unchanged too.
Where GitHub's rules for the parent branch require a pull request, an `off` landing that is no beam's
waits on it as before, and now gets the session as well.

Closes #282

## Design decisions

- The session ends a review with `lym loom pr-continue`, and `Finalize` merges as before.
  The session runs under the publish shell guard and its prompt, `landing-template-pr-review`,
  forbids it to push, approve, merge, close or reopen the pull request.
- `lym loom pr-continue` now admits the operator's shell, an orch,
  and the PR-Review session whose strand name the run's pull request record holds.
  It refuses every other agent session.
  The orch role contracts and `PATTERN-landing` now say a review ends only on the operator's word.
- The row reuses a live session or resumes the recorded one, and keeps its identity in the pull request record.
  A pause stops the session and the end of the pause resumes it.
  A session that ends while the review is open halts the row.
- The session is stopped before a read that may end the review, and resumed if the review stays open.
  It is stopped on a verdict, a halt or a non-transient error,
  and left running on a cancelled context or a transient error.
- A gate failure of a commit made during the wait is mailed to the session once per head, instead of halting the row.
  The row still halts when the mail cannot be posted, the review has already ended or the pull request is closed
  with no continue flag.
  A gate that judged no code, after a runner crash or a memory pressure stop, also halts the row and mails nothing.
- A re-entry past an ended review's gated head runs the gate before it pushes anything.
- `lym loom pr-continue` is also the retry for a gate failure that lies outside the change:
  it clears the mailed head so that the row gates it again.
  The same call sets the continue flag, so the review ends once a head passes.
- The `pr-reviewer` role in `agents.yaml` and the `landing.pr_reviewer` key (default `opus[high]`) are live again
  under their old names.
  The `agents-pr-reviewer-role-retired` reinterpretation is removed, so a hub keeps its own `pr-reviewer` entries.

## To check by hand

No live run of the session against real tmux, a real agent and a GitHub pull request is recorded.
Try one under `review`: the session appears in the task's window, answers a comment, commits a fix,
and ends the review with `lym loom pr-continue` when told to.

Review of this change found the problems below in the code it adds.
None of them describes the target branch, which has no PR-Review session.
Check each against the code:

- A task head the gate refused could pass an ended review as done, unpushed
  (`gateHead` and `gateBeforeRestart` in `internal/landingshed/prreview.go`).
  The row now halts there.
- A gate failure whose mail failed was recorded as delivered, so the wait went on silently (`gateHead`, same file).
  The row now halts and records no mailed head.
- Retrying a gate failure with `lym loom pr-continue` also ends the review
  (`gateRetryHint` in `prreview.go`, `internal/loomcli/prcontinue.go`, the session's prompt).
  This behaviour remains; the mail, the prompt, the command help and `docs/overview.md` state it.
- `lym loom pr-continue` admitted any `pr-reviewer` strand, including another run's
  (`prContinueAdmitsStrand` and `prContinueVerb` in `prcontinue.go`).
  It now admits such a strand only when its name is the recorded one.
- The session was stopped and resumed at polls where the wait could only go on
  (`sessionStopWanted` in `prreview.go`).
  It is now stopped only for a read that may end the review or act on its ending.
- The PR-Review session tests use module stand-ins, which `PATTERN-test-stand-ins` forbids a change to add.
  They run the real `shuttleengine.Runner`, but over `shuttlefake` with its new `ResumeEngine`,
  and with hand-written `SessionLive`, `RegisterSession` and `MailAgent` closures
  (`prreview_session_test.go`, `prreview_wait_test.go` and `prreview_test.go` in `internal/landingshed`,
  `internal/testkit/envkit`, `internal/testkit/shuttlefake`).
  This remains open.
- The stop before an ending read and the resume after it had no test.
  Check that the rows now in `internal/landingshed/prreview_session_test.go` pin both.
- No test pinned that `wireLanding` fills `SessionLive` and `RegisterSession`
  (`internal/loomcli/wiring_publish_integration_test.go`).
  Check the assertions added there.

## Merge danger

A revert does not undo all of it.
The pull request record, which reaches other machines, gains `mailed_head` and the session's identity fields.
A hub's `agents.yaml` may also come to hold a `pr-reviewer` entry, which the binary before this change refuses to load,
and a retirement reinterpretation already applied is not expected to run again and remove it.

**Door:** one-way
**Blast radius:** landing
