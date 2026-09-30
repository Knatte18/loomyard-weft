# Plan-Write respawn: deny notice, prior-plan block, rotator amendment

All six batches reached `done` and were committed.
The plan-level integration verify (build, `go test -tags integration`, `go vet -tags smoke`) ran once at HEAD `b9dff9e63` and reported `status: OK` with no deviations.

- 01 claudeengine-deny-notice: `buildSettings` now yields a one-line notice for each deny it installs.
- 02 claudeengine-append-system-prompt: the notice goes out via `--append-system-prompt` on launch and resume, with docs.
- 03 singlellm-prompt-amendment: the fresh-spawn preparation can append to the respawned session's prompt.
- 04 prior-plan-stencil: added the `loom-template-prior-plan` stencil and pointed the plan template at it.
- 05 loomengine-prior-plan-block: the prior-plan block renders from the stencil.
- 06 rotator-prior-plan-amendment: the Plan-Write rotator returns the block naming the archive it created.

`lyx webster status` also lists an earlier recovery entry for batch 01 (number -1) as stuck.
The batch-01 fork itself is `done`, so the run was not blocked.
