# Rename the lyx binary and the LYXHUB suffix (name clash with LyX)

`rename-binary-lyr` · Note · enhancement, stencil, shuttle

Imported from [issue #670](https://github.com/Knatte18/loomyard/issues/670).

`lyx` collides with LyX, the LaTeX editor, which ships as the `lyx` package in Ubuntu/Debian and installs `/usr/bin/lyx`.
On a machine with both, whichever comes first on PATH wins: a shell where `~/go/bin` is not yet on PATH starts the LaTeX editor instead of LoomYard.
LyX is common among researchers, so the clash is likely for other users too.

## Name

The new name is open: `lyd` and `lyr` are the candidates, and the operator has not chosen.
Neither is an Ubuntu package name (checked 2026-10-17 and 2026-10-26).
Rejected: `warp` and `ply` are Ubuntu packages, `weft` and `warp` are already fabric terms here, and the four-letter weaving names (`pirn`, `sley`) are longer to type.

## Priority (operator, 2026-10-26)

The rename lands before LoomYard is taken into use on other repositories, since renaming hubs and records already in use elsewhere is far more work.

## Scope

- The binary: the `cmd/lyx` build target and `binaryName()` in `tools/deploy`, docs and README, stencils and skills in the `ly` plugin that invoke `lyx`, and any scripts or launchers that call it.
- The hub directory suffix `-LYXHUB`, such as `loomyard-LYXHUB`, which the new name replaces (operator, 2026-10-26).

## Runs alone (operator, 2026-10-22)

The rename runs alone, with no other run in flight, since it touches nearly every file; it is promoted and started only then.
