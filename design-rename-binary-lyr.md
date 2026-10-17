# Rename the lyx binary to lyr (name clash with LyX)

`rename-binary-lyr` · Note · enhancement, stencil, shuttle

Imported from [issue #670](https://github.com/Knatte18/loomyard/issues/670).

`lyx` collides with LyX, the LaTeX editor, which ships as the `lyx` package in Ubuntu/Debian and installs `/usr/bin/lyx`.
On a machine with both, whichever comes first on PATH wins: a shell where `~/go/bin` is not yet on PATH starts the LaTeX editor instead of LoomYard.
LyX is common among researchers, so the clash is likely for other users too.

Operator decision 2026-10-17: rename the binary to `lyr` (LoomYard Runner), replacing the first proposal `lyd`.
Checked 2026-10-17: `lyr` is no Ubuntu package name, is not installed on the operator's machine, and a web search found no CLI tool whose binary is named `lyr`.
Rejected: `warp` and `ply` are Ubuntu packages, `weft` and `warp` are already fabric terms here, and the four-letter weaving names (`pirn`, `sley`) are longer to type.

Scope: the `cmd/lyx` build target and `binaryName()` in `tools/deploy`, docs and README, stencils and skills in the `ly` plugin that invoke `lyx`, and any scripts or launchers that call it.
