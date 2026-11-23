# Narrowed code_auto_accept reaches hubs only as a flag

`code-auto-accept-flag-only` · Note · bug, loom, config

Left open when `code-bouncer-asks-parent` landed (8cf7f4775), as finding `code-auto-accept-meaning-change-no-reinterpretation`.

The narrowed meaning of `code_auto_accept` reaches hubs only through a flagging reinterpretation in `internal/loomengine/reinterpret.go`, with no rewrite.
Confirm a flag is enough, or rewrite the hub layers that set the key.
