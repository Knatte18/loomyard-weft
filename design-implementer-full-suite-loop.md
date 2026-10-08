# Implementer forks run the full test suite on every edit

`implementer-full-suite-loop` · Note · bug, webster, stencil

## Seen

Before the 2026-10-08 crash, the `binary-change-and-rules` webster implementer forks ran `go build ./... && go test ./...` every 20 to 60 seconds, about 15 times between 19:58 and 20:06 CEST.
They also ran `go test -tags integration` and `go vet -tags integration ./...` across the whole module.
Each run starts up to `GOMAXPROCS` test binaries that link cgo and tree-sitter, and the integration tests start their own tmux servers, git and lyx children.
That ran alongside the other runs' gates, and the user slice ran out of memory.

## Idea

An implementer fork verifies only the packages its batch touches, and only once per change, never the full module on each edit.
The full suite belongs to the plan-level verify gate.
The webster stencil and the batch report state which packages a fork may test.
