# Rename the `lyx` binary to `lyd` (name clash with LyX)

`rename-binary-lyd` · Note · enhancement, stencil, shuttle

Imported from [issue #670](https://github.com/Knatte18/loomyard/issues/670).

`lyx` collides with LyX, the LaTeX editor, which ships as the `lyx` package in Ubuntu/Debian and installs `/usr/bin/lyx`.
On a machine with both, whichever comes first on PATH wins: a shell where `~/go/bin` is not yet on PATH starts the LaTeX editor instead of LoomYard.
LyX is common among researchers, so the clash is likely for other users too.

Proposal: rename the binary to `lyd` (LoomYard driver).
It is short, close to `lyx`, and not an Ubuntu package name.

Operator view, not a decision (2026-10-17): not quite happy with `lyd`; the operator has thought of `lyr`, for "LoomYard Runnable" or "Runner".
Checked 2026-10-17: neither `lyr` nor `lyd` is an Ubuntu package name or installed on the operator's machine, and a web search found no CLI tool whose binary is named `lyr`.

Scope: the `cmd/lyx` build target and `binaryName()` in `tools/deploy`, docs and README, stencils and skills in the `ly` plugin that invoke `lyx`, and any scripts or launchers that call it.
