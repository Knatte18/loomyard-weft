---
format: 5
approved: false
language: go
---

# Plan: a gate run keeps the agent's prompt cache warm

An agent blocked on `lym gate test` or `lym gate card-verify` gets an answer within its keep-warm cap even when the run outlasts it: a capped call starts or joins a detached runner, answers `done: false` with exit 75 when its cap passes, and the same command again prints the run's result once it ended.
The cap moves from the constant `shuttleengine.DefaultAwaitCap` to claude.yaml config per role, answered through the engine seam and one resolver in `internal/shuttleengines`, which the gate and `lym shuttle await` read; a `--gate` key whose runs outlast the break-even waits in one long call.
Gate calls rendered for agents drop `--wait`, every `verifyrun` caller runs its command without the strand, and the agents' text reads 75 with `done: false` as "run again".

## Card Index

1 — shuttle-max-reruns-key — add shuttle.yaml's keep_warm_max_reruns break-even
2 — engine-keep-warm-caps — claude.yaml keep-warm caps, their validation and the engine's KeepWarmCaps method
3 — keep-warm-resolver — shuttleengines.KeepWarm resolves a strand's cap with a fallback
4 — await-resolved-cap — lym shuttle await defaults to the resolved cap; texts say 3600000 ms
5 — strand-free-verify-commands — verifyrun callers run commands without the strand
6 — verifyrun-tied-line — verifyrun.RunTied ties a line's shell to its parent and reports its group
7 — cardverify-context-and-tie — cardverify.Runner takes a context and reports its phase and line tree
8 — runproc-gate-role — add runproc.RoleGate
9 — run-records-call-id — run records carry a call id and an abandoned mark, counted apart in the report
10 — gate-detached-runner — the hidden run-detached verb, its run directory and its lease
11 — gate-capped-front — the capped front of lym gate test: choice, join, start, answer
12 — gate-long-wait — a --gate key past the break-even waits with the long cap
13 — card-verify-capped — lym gate card-verify takes the capped form
14 — agent-gate-calls — gate calls rendered for agents carry no --wait
15 — agent-keep-warm-text — the gate rule, the implementer stencil and the darn and drift stencils read done: false as run again

## Shared Decisions

### Decision: one cap source

- **Decision:** a cap reaches `internal/gatecli` and `internal/shuttlecli` only as `shuttleengines.KeepWarm`'s answer; `DefaultAwaitCap` stays only as that resolver's fallback, and `cmd/testtiming/resources.go` keeps it as decided Out of scope.
- **Rationale:** PATTERN-shuttle-provider-seam and the declared `PATTERN-keep-warm-cap`.
- **Applies to:** cards 3, 4, 11, 12, 13

### Decision: tests replace the bounds, never wait them out

- **Decision:** the front's cap, lock margin, claim bound, grace and launcher are `detachSeams` fields, and the gate helper sets them from its environment; production launch refuses under `testing.Testing()`.
- **Rationale:** PATTERN-test-speed and the Constraint that tests never re-exec `os.Executable()` under `go test`.
- **Applies to:** cards 10, 11, 12, 13

## PATTERN changes

- `PATTERN-keep-warm-cap` — adds: Bounding a wait an agent blocks on: the cap from `shuttleengines`' one keep-warm resolver for the caller's role, never a constant bar its fallback; a long gate run goes on in a detached runner the same command joins.
