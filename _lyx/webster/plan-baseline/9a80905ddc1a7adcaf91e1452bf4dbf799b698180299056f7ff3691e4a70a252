---
format: 5
approved: true
language: go
---

# Plan: hibernate-trigger

Let the operator, through the orch, queue a hibernate of the machine that lyx's own Go code carries out once every session in the hub is safely paused, so no LLM session runs the hibernate and none is cut mid-call.
A new `hibernate` module (`internal/hibernateengine` over seams, `internal/hibernatecli` wiring it to the hub) records one request per hub and spawns a detached daemon that waits for its trigger, pauses the hub through tender's engine, confirms the pause, runs the hibernate, detects the wake and lifts its own pause entry.
Tender's engine gains a lift of one entry by identity and a callback naming the entry a pause adds; spawned sessions are denied the machine sleep commands; `lyx p` and `lyx r` join the shortcuts; and the orch's role file names the new verb.

## Card Index

1 — tender-lift-entry — tenderengine lifts one pause entry by identity and tells a pause's caller the entry it added
2 — tendercli-export-hubdeps — export tendercli's hub wiring of the pause engine as HubDeps
3 — hibernate-engine-verbs — the hibernate request file and the submit, cancel and status sequences
4 — hibernate-engine-watch — the daemon's trigger wait, pause, confirmation, hibernate and wake
5 — hibernate-cli — the lyx hibernate module, its hub wiring and spawns, and its docs and guards
6 — sleep-deny — deny the machine sleep commands to every spawned Claude session
7 — shortcuts-p-r — the lyx p and lyx r shortcuts for pause and resume
8 — orch-role-hibernate — the orch role file routes an operator's hibernate request through lyx hibernate

## Shared Decisions

- The engine reaches tender only through the `Pause`, `TenderStatus` and `Lift` seams of `hibernateengine.Deps`; the CLI fills them from `tendercli.HubDeps` with the caller label `hibernateengine.PauseCaller`.
- The request file and the daemon's log live in `<hub>/_board/.lyx/hibernate/`, through `lyxdirs.HibernateDirName` and `fabricengine.HubScratchDir`.
- No test runs `systemctl`, spawns the daemon or waits out a production duration: engine tests drive fakes and an injected wall and awake clock.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
