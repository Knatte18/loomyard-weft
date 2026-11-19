# Verify triage lists verify-command as a flaky identity with no failure output

`verify-triage-command-identity` · Note · bug, webster, gate

Imported from [issue #160](https://github.com/Knatte18/loomyard/issues/160).

The integration-suite triage after a `lym webster` verify run wrote a friction note listing `verify-command` as a non-regression failure that "failed the first verify run and passed or vanished on rerun (flaky)".
The only other line in the note is "Shell cwd was reset to <worktree root>", which is the harness's cwd-reset notice, not a test result.

This is abnormal because the identity `verify-command` names the verify step itself rather than a test.
A failure of the step itself, possibly caused by the cwd reset, is classified as a flaky test, so a lym-side verify problem is hidden as test flakiness.
The note does not say what the first run's failure was, so the cause can't be recovered from it.

What to do:
- Check how triage derives identities and whether `verify-command` can stand for a failure of the command rather than of a test.
- Keep the first run's failure output in the triage note.
- Keep a cwd-reset notice out of the failure identity and the note body.

Provenance: task `miqp-baseline-oracle`, note `webster-verify-triage.md`.
