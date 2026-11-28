# testkit: replay tmux pastes, and test PR-Review mail over real creel

Replaces hand-written stand-ins with real modules: the `MailAgent` closures of the PR-Review mail tests,
and `envkit`'s PR-Review session seams, sampler session seams and live-strand seam.
The closures meant no real creel path ran, and one of them failed with a made-up "no such strand" error.
The recorded tmux kit (`internal/testkit/tmuxrec`) also learns to replay a paste, which it could not do before.

## What changes

**PR-Review mail rows.**
The three mail rows leave `TestPRReview_Wait` in `landingshed`, which cannot import creel's client without a cycle.
They are now the tier1 test `TestPRReviewGateFailureMail` in `internal/loomcli/prreview_mail_integration_test.go`.
It drives the row through the mail seam `loomcli` wires, posting to creel's daemon run in process on a fake delivery.
The rows pin one mail per refused head over three wakes, and a cleared mailed head gated and mailed again.
They also pin that creel's refusal of an unknown strand halts the row with no mailed head recorded.
The shared `landingshed` test deps now fail the test on any `MailAgent` call, so the rows that mail nothing still check it.

**Paste replay in `tmuxrec`.**
A recorded call now keeps its standard input, and a replay matches on it.
A recording holds `{inject-buffer}` in place of a paste's random buffer name.
A replayed call matches whatever name it draws, as long as it names only one.
A new `session-paste` recording pastes two lines into a session's pane, and the tier0 replay runs over it.

**`envkit` sessions.**
The PR-Review and sampler session seams now run a real session runner over real reed,
whose tmux replays the `session-start-stop` recording.
Their liveness and registration seams, and `FullEnv`'s live-strand seam, read that reed's strand table.
`envkit.LandingDeps` now takes the test as its first argument.

**Production code.**
`reedengine` exports `InjectBufferPrefix` and `LiveStrandNames`.
`loomcli`'s wiring calls `LiveStrandNames` instead of its own inline closure, with the same behaviour.

## Design decisions

- In a recorded paste's text only the hub directory becomes a placeholder.
  Numbers and other bound values can be literal text the code under test sent, so they stay as sent.
- `tmuxrec.IdleSession` is for kits that build session seams without driving them.
  It never skips; on an OS other than the recording's, a call it makes goes unanswered and fails.
- A host lacking the configured shell now still builds a replayed session, binding the shell's bare name.
- `envkit`'s other hand-written runners, such as `noopFixerSeats` and the `shedfake` ones,
  and its batten child seams stay as they were.
- The moved mail rows no longer check that the wait mark is set and cleared.
  The rows that stay in `TestPRReview_Wait` still check it.

## To check by hand

- `internal/loomcli/prreview_mail_integration_test.go`, `prReviewMailDeps`:
  the dirty-path, missing-commit, task-commit, pause and record-watch seams, among others,
  come from `loomcli`'s wiring over the real task pair.
  `CommitRecord`, `BaseCommand` and `GateTiers` keep `envkit`'s hand-written answers.
  The wired `CommitRecord` pushes to the pair's origin, a GitHub URL here,
  and the wired gate seams drop the tier from the mail and halt reason on this fixture hub.
  Decide whether those three answers are acceptable.
- Same file: the new test adds uses of two standing fakes, `prkit.API` and `creelfake.Delivery`.
  `pattern/PATTERN-test-stand-ins.md` says no change adds a stand-in it forbids,
  and the inventory (`docs/research/no-fake-other-modules/`) gives `prkit.API` the verdict `goes`
  and `creelfake.Delivery` the verdict `real`.
  The unknown-strand refusal therefore rests on the fake delivery's hand-written strand list.
  The change description the test writes by hand at `deps.DescriptionPath` is also a hand-written stand-in.
  Decide whether these are acceptable here.

## Merge danger

**Door:** two-way
**Blast radius:** reedengine
