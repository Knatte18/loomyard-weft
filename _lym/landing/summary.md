# Run PR-Review mail over real creel and teach recorded tmux to paste

The PR-Review session tests and `envkit` still had stand-ins for repository modules.
The mail seam in the PR-Review wait tests was a closure, and one row failed it with a made-up "no such strand" error.
`envkit` filled the session seams with a runner over a hand-written reed fake and a sampler type that did nothing.
The recorded tmux kit could not take their place where text is sent:
it neither kept nor matched a call's standard input, and a paste names a fresh random buffer each time,
so a replayed paste never matched its recording.

## What changes

- `internal/testkit/tmuxrec` records the text a tmux call reads on standard input and matches a replayed call on it.
  A recording holds `{inject-buffer}` in place of a paste's buffer name.
  A replayed call matches whatever buffer name it draws, as long as it names only one.
  The buffer prefix is now `reedengine.InjectBufferPrefix`, which reed and the kit both read.
- A new `session-paste` recording, made over tmux 3.6 on Linux, pastes two lines into a session's pane.
  The tier0 replay runs over it, and new rows pin that a paste matches by its text and not by its buffer name,
  and that a call naming two buffers is refused.
- `tmuxrec.SessionSpec` exports the spec every recording starts under.
  `tmuxrec.IdleSession` builds a replayed session that never skips, for a kit whose tests build session seams but do not drive them.
- `envkit`'s PR-Review and sampler session seams now run a real `shuttleengine.Runner` over real reed on the `session-start-stop` recording.
  Their liveness and registration seams read that reed's strand table through `reedengine.SessionLiveness` and `reedengine.SessionRegistrar`.
  `envkit.LandingDeps` now takes the test.
- The three PR-Review mail rows move to a tier1 test, `internal/loomcli/prreview_mail_integration_test.go`.
  It drives the row through the mail seam loomcli wires, posting to creel's daemon run in process.
  The gate runs a failing command in a real task pair.
  The rows pin one mail per refused head over three wakes, a cleared mailed head gated and mailed again,
  and creel's own refusal of an unknown strand halting the row with no mailed head recorded.
- In `landingshed`, the shared PR-Review deps' mail seam now fails the test when called.
  So the rows that must mail nothing (a closed pull request, an ended review, a runner crash) still check that, with no mail stand-in.

## Design decisions

- The mail rows live in `loomcli` because `landingshed` cannot import creel's client: `creelcli` depends on `landingshed`.
- The tier1 test seeds a live session and names the strand in the record itself.
  A session the row started would carry the recording's `tr` shortname, which the test hub's creel does not serve.
- The tier1 test takes its landing deps from `envkit.LandingDeps` and replaces the seams the wait reads.
  The kit's other closures stay as they are, a standing fake the stand-ins inventory already lists.
- No session in `envkit` sends text, so none of its seams needs the paste replay.
  The paste recording and its replay test stand in `tmuxrec` for the session tests that will send text.
- A recording keeps the numbers in a paste's text as they are.
  Only process ids that tmux handed out become placeholders, and the text is what the code under test sent.
- A host without the configured shell now builds a replayed session with the shell's bare name bound,
  instead of failing every test that builds one.
  Reed's first start on such a host fails, as it would in production.

## To check by hand

- `IdleSession` does not skip off Linux, so an `envkit` test that drove its sessions there would fail with an unanswered call.
  No test drives them today.

## Merge danger

**Door:** two-way
**Blast radius:** testkit
