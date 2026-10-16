---
format: 5
approved: false
language: go
---

# Plan: creel stage 1 re-test fixes

Fix the eight creel defects of the 2026-10-16 creel stage 1 re-test, #677 first: the journal tail repair (#677), the stderr fallback of a refusal whose stdout cannot be written (#681), stall freshness (#682), the client's wait on a spawned daemon that exited (#683), carried records in `log` (#684), the removed-holder sweep timing (#685), the no-input-box stall and TTL-capped probe waits (#686), and the llm smoke test with its engine-neutral startup-gate seam (#678).
Each card carries its tests and the docs its change makes wrong: the package `doc.go` files, the creel lines of `pattern/PATTERN-daemon-wakeups.md` and the C4, C6, C8, C10 and C11 watch lines of `tools/sandbox/SANDBOX-CREEL-SUITE.md`.

## Card Index

1 — journal-tail-repair — cut a torn last journal line whether or not it ends in a newline, sharing the fold's last-line rule
2 — refusal-stderr-fallback — write a creel refusal to stderr when its stdout envelope cannot be written
3 — stall-freshness — give a stall to a waiter or an expired answer only from a reading no older than twice the probe ceiling
4 — spawn-exit-signal — end the client's connect wait when its spawned daemon exits other than by a lost start race
5 — carried-log-entries — list each carried record as a carried log event, after a fold and in a running daemon
6 — sweep-confirm-timing — confirm a first miss after 10 s, cap the sweep backoff at 50 s and never lengthen it on new mail
7 — idle-probe-no-input-box — report a pane with no input box on shuttle's idle probe
8 — no-input-box-stall — name the no-input-box stall in creel from the probe field
9 — ttl-capped-probe-wait — cap a live receiver's probe wait at its soonest inject's remaining TTL
10 — startup-gate-seam — add shuttle's engine-neutral seam that accepts a provider's startup gate on a live strand
11 — llm-smoke-strand — place the llm smoke strand in its own window, accept its startup gate and print its pane on failure

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
