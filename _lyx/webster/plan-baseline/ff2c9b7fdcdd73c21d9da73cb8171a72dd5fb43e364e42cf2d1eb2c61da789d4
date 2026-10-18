---
format: 5
approved: true
language: go
---

# Plan: creel stage 2 — notices and the orch's typed lines go through creel

Every notice and every line lyx types into the orch session goes through creel.
Creel delivers into orch strands and gains command injects, a Go-only withdraw verb and a typed refusal code; its reminder stops carrying content.
Notices from batten, loom, burler, webster, hibernate and shuttle become creel mail through `orchcli.NotifyPrime`, tender's orch revive becomes a checked inject, and the hub notice log goes.
The orch watcher keeps only its cycle logic: it reaches creel through a seam, posts its note request as mail answered by post, delivers every step as a creel inject confirmed by creel's outcome, and holds the orch through a creel hold; its notice queue and its idle probe before typing go, and `lyx orch start` withdraws the old cycle around a fresh launch.

## Card Index

1 — shuttle-event-time — give each shuttle event its hook-side time from the session-signal reading
2 — creel-orch-receiver — deliver into orch strands, hold rules on them, a reminder without subject
3 — creel-command-inject — command injects from a program to an orch strand through a new Delivery operation
4 — creel-withdraw-and-refusal-code — Go-only withdraw verb, the outside-pane refusal code and a bounded client call
5 — creelcli-go-client — the exported Go client other modules call creel through, and command injects in the log
6 — orch-notify-over-creel — NotifyPrime posts creel mail and every notice caller moves to it
7 — tender-orch-revive-inject — tender's orch revive as a checked creel inject under the record lock
8 — drop-hub-notice-log — remove batten's notice log and its paths, with the durable-state PATTERN change and the overview room both PATTERN changes need
9 — orch-creel-seam — the watcher's creel seam, the pause-resume line as mail, the notice queue removed
10 — orch-cycle-over-creel — the watcher's cycle delivered and confirmed through creel
11 — orch-start-withdraws — orch start's two withdraw rounds and the watcher's retry of a failed one

## Shared Decisions

### Decision: note-request-id

Creel assigns a message's id when the post is journaled, so the note request's body cannot carry its own id as a literal.
The body tells the orch to reply to the id `lyx creel read` prints beside that message (`lyx creel post --reply <id> "written <path>"`, or `defer` for a soft request), and the watcher records the id the post answers and gates on a reply whose `reply_to` is that id.
This is how card 10 meets D4's "the body names the message id to reply to".

### Decision: go-client-surface

`creelcli.GoClient` (card 5) is the one Go-side caller of creel: `NewProgramClient(location, program)` signs as `<shortname>:<program>` with no strand claim, `NewStrandClient(location, name)` claims a strand with no program.
Each sending module owns its program name as a constant (`BATTEN`, `LOOM`, `BURLER`, `WEBSTER`, `HIBERNATE`, `TENDER`, `ORCHWATCH`), formed into the full name through `agentname.FormatProgram` inside the client.
Every client call carries `goCallTimeout`, which card 4 makes bound every request, so the creel lookups tender makes under the pause record lock are capped (D7).

### Decision: notify-answer

`orchcli.NotifyPrime(location, program, line)` answers `(posted bool, err error)`, so batten's `Notify` seams keep reporting whether a notice went out.
A recorded orch GUID is the test of "recorded", as batten's and loom's gates keep it; a GUID the prime's reed state no longer holds is addressed by the fixed `<shortname>:orch` name, and creel decides whether that strand is listed (`orchcli.PrimeOrchStrand`).

### Decision: watcher-addresses-by-name

The engine's creel seam takes receiver names; the watcher resolves the orch strand's name from the recorded GUID through `Session.StrandName` at each call, and `lyx orch start` uses the listed strand's name or else the fixed `<shortname>:orch`, which `withdraw` and the second `release` accept whether or not creel lists the strand.

### Decision: bind-color-in-memory

The bind color's inject id is watcher memory only; a restarted watcher may queue the color once more, which is display only.

## PATTERN changes

- `PATTERN-orch-pane-single-writer` — changes: Typing into the orch session from Go: only creel, through its `Delivery` seam; the orch watcher keeps only a hub pause's guarded interrupt, reed's `/rename` repair keeps its own guard, and other modules post or inject through creel.
- `PATTERN-durable-vs-ephemeral-state` — changes: Adding a file: tracked under `_lyx`, never-tracked under `.lyx`, same subpath; nothing a run, the orch or a landing depends on lives only under `.lyx`. The background file gains, verbatim: "Nothing a run, the orch or a landing depends on lives only under `.lyx`: decisions, landings, stop reports and board state are always in git (board, records, decision record), and never-tracked state such as the creel journal is a delivery path or cache that a fresh machine starts without."

## verify:

go build ./cmd/lyx
lyx gate test ./cmd/lyx ./contracts/stencils ./internal/shuttleengine ./internal/testkit/shuttlefake ./internal/creelengine ./internal/creelcli ./internal/testkit/creelfake ./internal/orchengine ./internal/orchcli ./internal/battencli ./internal/battenshed ./internal/loomcli ./internal/burlercli ./internal/webstercli ./internal/hibernatecli ./internal/tenderengine ./internal/tendercli ./internal/shedrun ./internal/testkit/envkit ./internal/landingshed ./internal/sheddriver ./internal/pattern
lyx gate test --tags integration ./cmd/lyx ./contracts/stencils ./internal/shuttleengine ./internal/testkit/shuttlefake ./internal/creelengine ./internal/creelcli ./internal/testkit/creelfake ./internal/orchengine ./internal/orchcli ./internal/battencli ./internal/battenshed ./internal/loomcli ./internal/burlercli ./internal/webstercli ./internal/hibernatecli ./internal/tenderengine ./internal/tendercli ./internal/shedrun ./internal/testkit/envkit ./internal/landingshed ./internal/sheddriver ./internal/pattern
go vet -tags llm ./internal/creelcli ./internal/orchcli
go vet -tags tmux ./internal/creelcli
