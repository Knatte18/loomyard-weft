# Test git can reach the network and open a credential prompt, hanging an agent's test run

`test-git-hermetic-network` · Note · bug, testkit, git

Imported from [issue #299](https://github.com/Knatte18/loomyard/issues/299).

## What happened

During a code-review fix round, an agent wrote an integration test that took its dependencies from `internal/loomcli`'s production wiring.
The test thereby ran the real record commit and push over a fixture pair whose origin is a github.com URL.
git asked for credentials through the desktop askpass helper, which opened a dialog on the operator's desktop.
The run hung for about eighteen minutes, until the test timeout.

## Why it is abnormal

A test run, unattended in an agent session, must neither reach the network nor prompt the operator.
The shared hermetic git environment (`internal/gitkit/hermetic.go`) stops neither, so any test that wires a real push can repeat this.
The one test that hit it now sets the variables itself, which protects no other test.

## What to do

Set these in the shared hermetic git environment, for every test:

- `GIT_ALLOW_PROTOCOL=file`
- `GIT_TERMINAL_PROMPT=0`
- `GIT_ASKPASS` set to empty

Check whether `SSH_ASKPASS` needs the same treatment, and whether any test relies on a non-file protocol.

Provenance: task `pr-session-mail-stand-ins`, note `burler-code-r2.md`.
