# lyx shuttle open cannot choose the permission mode

`shuttle-open-permission` · Task · enhancement, shuttle · running · Finalize

## Seen

On 2026-10-15 the orch opened a dedicated Opus session for the operator with `lyx shuttle open --role stencil-audit`.
It came up in prompt mode, since an interactive run resolves an empty `PermissionMode` to prompt (`claudeengine.validatePermissionMode`), and `shuttle open` has no way to set it.
The operator wanted bypass permissions; the orch removed the strand and started a plain `claude --dangerously-skip-permissions` through `lyx reed add`, which then registered under a generic session name (`loomyard-13`) instead of `ly:stencil-audit`.

## Idea

`lyx shuttle open` takes a `--permission-mode` flag (bypass or prompt), or a config default, so an operator session opens in bypass with its lyx strand name.

## Operator decision (2026-10-15)

Build it as a small task, from the 2026-10-15 board triage, once `creel-stage1-fixes` has landed, since both touch the shuttle module.