# A running verify gate is visible in status and on screen

`gate-running-visible` · Note · enhancement, loom, reed

Seen 2026-10-05 in `orch-cycle-hardening`: after the Webster-Burler agent ended its turn, the producer ran the task's verify command (`verifyrun`: build, unit, integration, smoke vet) for many minutes.
Nothing on screen said so: the agent's pane sat idle, `lyx loom status` showed only `now=Webster-Burler`, and the operator read it as a hang, asked the agent what it was waiting on, and the agent could not tell either.

Idea (operator): a visible marker whenever a Go gate runs.

- The status (`activity.now`, and the `loom-status` strand) names the gate and how long it has run, such as `Webster-Burler: verify running 6m (go test -tags integration)`.
- The agent pane, or its strand's title, shows that the run moved on to a gate, so an idle agent does not look stuck.
- The same applies to every Go-side wait a producer does after its agent's turn: the Webster verify gate, the Burler round's verify, Publish's merge-in verify.
