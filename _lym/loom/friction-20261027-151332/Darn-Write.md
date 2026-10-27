# Darn-Write handoff re-prompted on a verify failure the run may not fix

The handoff verify failed on `TestEnforcement_MarkdownLinks` every round, and every broken link was in `main`'s bench prompt captures under `docs/research/*/data/`, which this change does not touch.
The parent's `Added after Discussion` decision forbids fixing them in this run and waits on `docs-link-skip-bench-data` landing on `main`, yet the run was resumed at Darn-Write before that task landed.

Each re-prompt said "fix every finding", which the recorded decision forbids, so each round could only end unchanged until the gate exhausted.
A resume that waits on another task landing could check that it has landed first.
