---
format: 5
approved: false
language: go
---

# Plan: tender — hub-wide control over what runs

A `tender` module takes over the pause module and adds hub-wide control: `lyx tender pause`, `resume`, `revive` and `status`, with `lyx pause` and `lyx resume` kept as shortcuts.
`revive` gives one carry-on line to each session stalled on an API error, through a conditioned shuttle send for task sessions and a conditioned orch notice for the orch.
`status` shows the pause entries and holds, every live session in the hub and the lyx processes running a replaced binary.
A pause no longer uses up the deadline of a GitHub call it holds (#606).
`lyx orch watch` re-execs itself onto a replaced binary at a restart-safe tick, and on Unix `lyx reed attach` replaces itself with tmux.
The package rename from `pausecli`/`pauseengine` to `tendercli`/`tenderengine` is the last card, so every earlier card names glyphs that exist in the tree today.

## Card Index

1 — pauserecord-network-held — `pauserecord.Network` reports how long it held the call
2 — github-held-deadline — the GitHub transport owns both deadlines and moves a held call's deadline by the held time
3 — github-per-request-deadlines — several-request GitHub callers derive a fresh deadline per request
4 — shuttle-conditioned-send — `Runner.SendConditioned`: at-once refusals, no clears, a bounded turn-start wait
5 — orch-conditional-notice — a conditioned orch notice, de-duplicated at queue time and dropped at delivery once its state no longer reads
6 — orch-watch-binary-reexec — the orch watcher re-execs onto a replaced binary only at a restart-safe tick
7 — proc-process-listing — `proc.ListProcesses` reads pid, start time, exe and cwd links and argv
8 — tender-command — one `tender` root with pause, resume and status; `pause --status` goes; pause and resume become shortcuts
9 — tender-status — status lists every live session in the hub and the processes running a replaced binary
10 — tender-revive — `lyx tender revive [<slug>]` under the pause record lock
11 — reed-attach-exec — `reed attach` execs tmux on Unix; the shortcut guard keeps attach last
12 — tender-rename — rename `pausecli`/`pauseengine` to `tendercli`/`tenderengine`

## Shared Decisions

- The revive line and the conditioned send's condition, idle-stalled with cause `api-error`, are declared once: the line in `pauseengine` (later `tenderengine`), the condition type in `shuttleengine` and the orch notice condition in `orchengine`.
- Tender never types into the orch's pane; the orch watcher stays its one writer.
- Every new reading or delivery is a seam the engine is told: the process reading, the session enumeration, the conditioned send and the notice queue (`PATTERN-told-geometry`).

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
