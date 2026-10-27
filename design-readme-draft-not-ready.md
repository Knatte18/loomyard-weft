# Board README lists draft tasks under Ready

`readme-draft-not-ready` · Task · bug, board · running · Finalize

Imported from [issue #873](https://github.com/Knatte18/loomyard/issues/873).

Seen (2026-10-25): a released-pending task with status draft and no open dependencies (codex-astra-solo-bench) renders under Ready, 'waits on nothing open; can start now', with only a draft tag. ComputeLayers (internal/boardengine/layer.go:33-41) handles done, running, project and isolated entries but not draft, so it falls into readyLayer. Expected: a draft task is not Ready; it gets its own layer (Draft) or sits apart from Ready, since dobby will not start it until released.

The board README rendering is not regarded as production code (operator, 2026-10-27).
