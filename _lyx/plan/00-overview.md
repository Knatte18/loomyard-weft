---
format: 5
approved: true
language: go
---

# Plan: pane-inject-safety

Text lyx types into a Claude pane never merges with, submits or wipes an operator's draft, and never closes a menu the operator has open.
Reed gains an atomic paste-submit and an operator-activity query; shuttle guards every write with one fresh reading of readiness, activity and an empty, readable box, pastes in one tmux invocation, sends an Enter of its own only into a box holding exactly the sent text, and records a send that may have left text in the box so its retry types nothing twice.
Claude drops its leading Escape and takes the paste path; the orch watcher, loom's driver resume and batten treat a held write as "later"; batten status flags a halted batten over a done child, and the orch's after-landing routine closes it.

## Card Index

1 — reed-paste-submit — add reed's one-invocation paste-submit transport op
2 — reed-operator-activity — add reed's operator-activity query and the operator_quiet_s key
3 — reed-name-repair-guard — put the watchdog's /rename behind the activity check and the paste-submit
4 — shuttle-reed-seam — widen shuttle's ReedOps seam and both fakes with the two reed ops
5 — shuttle-prewrite-guard — hold a write for an active operator and for an unreadable input box
6 — shuttle-paste-path — paste through reed for a TextPaster engine and send an Enter only on an exact match
7 — shuttle-pending-send — record a send that may have left our text in the box and dedupe its retry
8 — shuttle-session-commands — send /clear, /compact, /reload-plugins and /color through the guarded path
9 — claude-paste-path — move Claude onto the paste path, drop the leading Escape, add the PATTERN entry
10 — claude-paste-live-check — pin Claude Code's bracketed-paste behaviour in an llm-tier test
11 — orch-held-typed-steps — make the orch watcher wait out a held write instead of failing the tick
12 — shedrun-driver-busy — declare the driver_busy kind and the resume report file in shedrun
13 — loom-driver-busy-refusal — refuse a busy driver pane with driver_busy and reuse one report path per park
14 — battenshed-driver-busy-retry — retry a busy driver pane and notice the orch once per stall
15 — battencli-driver-busy-kind — map the child's driver_busy refusal to batten's sentinel
16 — batten-status-halted-flag — flag a halted batten whose child loom run is done
17 — orch-after-landing-stencil — make the orch's after-landing routine close batten

## Shared Decisions

### Decision: settle-only-for-the-no-enter-variant

- **Decision:** `awaitSettledBox` and `submit_settle_ms` stay only where an Enter follows typed or pasted text in a separate step: the no-Enter paste variant and a typing engine's box path. The paste-submit with its trailing Enter needs no settle, since tmux delivers the paste and the Enter in one command list.
- **Rationale:** the settle exists to keep an Enter out of a typing burst; a bracketed paste with an explicit end has no burst for it to wait out.
- **Applies to:** cards 6 and 9

### Decision: set-buffer-over-load-buffer

- **Decision:** reed's paste-submit fills its named buffer with `set-buffer -b <name> -- <text>`, not `load-buffer` from stdin.
- **Rationale:** the transport keeps `TmuxCmd.run`'s argv-only shape and its test hook; every text lyx injects is one short line, far below tmux's message limit, and a longer one fails loudly with nothing pasted.
- **Applies to:** card 1

### Decision: pending-send-file

- **Decision:** the pending-send record lives in its own `pending-send.json` in the run directory, not as a `run.json` field.
- **Rationale:** `run.json` has writers in the run loop and in finalize; a file of its own keeps the record's writes from racing them.
- **Applies to:** card 7

### Decision: report-path-keyed-to-the-park

- **Decision:** the resume branch's stored report path is reused only while the park marker's content, the stop report the driver parked on, is the one it was picked under.
- **Rationale:** the driver removes the marker itself on a self-initiated re-step and leaves the resume-report file behind; keying the file to the marker's content makes a new park pick a new path without a second remover.
- **Applies to:** card 13

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
