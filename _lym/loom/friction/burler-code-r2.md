# Burler fix, code round 2: the PR-Review mail test

A trial that took every dep from loomcli's wiring gave a test the real record commit and push over a pair whose origin is a github.com URL.
git asked for credentials through the desktop askpass helper, which opened a dialog on the operator's desktop and hung the run for about eighteen minutes until the test timeout.

Nothing in the test environment stops a test's git from reaching the network or prompting.
Setting `GIT_ALLOW_PROTOCOL`, `GIT_TERMINAL_PROMPT` and `GIT_ASKPASS` in the shared hermetic git environment would stop it for every test.
