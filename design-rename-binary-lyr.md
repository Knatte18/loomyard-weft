# Rename the lyx binary and the LYXHUB suffix (name clash with LyX)

`rename-binary-lyr` · Note · enhancement, stencil, shuttle

Imported from [issue #670](https://github.com/Knatte18/loomyard/issues/670).

`lyx` collides with LyX, the LaTeX editor, which ships as the `lyx` package in Ubuntu/Debian and installs `/usr/bin/lyx`.
On a machine with both, whichever comes first on PATH wins: a shell where `~/go/bin` is not yet on PATH starts the LaTeX editor instead of LoomYard.
LyX is common among researchers, so the clash is likely for other users too.

## Name

The new name is open, and the operator has not chosen.
The note `binary-name-candidates` holds the clash-checked candidates: short, preferably three letters, ideally rooted in weaving, with `vef` ranked first and `vev` close behind.
Rejected: `ly`, since the binary and the `ly` plugin must not share a name; `loy`, too close to `ly`; and `lyd` (operator, 2026-10-26).

## Priority (operator, 2026-10-26)

The rename lands before LoomYard is taken into use on other repositories, since renaming hubs and records already in use elsewhere is far more work.

## Scope

- The binary: the `cmd/lyx` build target and `binaryName()` in `tools/deploy`, docs and README, stencils and skills in the `ly` plugin that invoke `lyx`, and any scripts or launchers that call it.
- The hub directory suffix `-LYXHUB`, such as `loomyard-LYXHUB`, which the new name replaces (operator, 2026-10-26).

## Done mechanically (operator, 2026-10-26)

The rename is done by a script, never by hand edits file by file.
It needs agents to be allowed scripted edits, which `scripted-edits-allowed` brings.

## Runs alone (operator, 2026-10-22)

The rename runs alone, with no other run in flight, since it touches nearly every file; it is promoted and started only then.
