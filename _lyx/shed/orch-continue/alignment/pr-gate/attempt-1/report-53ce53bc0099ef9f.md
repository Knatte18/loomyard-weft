---
verdict: APPROVED
---

# Alignment at PR-Gate: `orch-continue`, pull request 669 at `4158bfa40`

## Goal against the change

The board entry asks that `lyx orch` record its session id and offer `lyx orch start --continue` to adopt the last orch session after a reboot, instead of the operator digging the id out of `~/.claude/projects/` for `--adopt`.
The diff does that and nothing beyond it:

- `internal/shuttleengine` adds `RunSessionState.SessionID` and `Run.SessionID()`, additive to the read-only reading (D5); `readSessionFacts` now shares `newestSessionID` with the reading, with unchanged behaviour.
- `orchengine.State` gains an optional `session_id` (D4, `PATTERN-tolerant-records`); the watcher records the id it reads each tick through the strand-checked save, and writes nothing when it is unchanged or unreadable.
- `start` records the launch's id at every fresh launch, clears it in `adoptStrand` when it binds another live strand, and takes the adopt branch with the recorded id under `--continue` (D2, D4).
- `--continue` refuses alongside `--adopt` or `--handoff`, with a live strand, and with no recorded id, each before any strand is removed or launched; a launch error under `--continue` carries the two ways forward (D3).
- A fresh launch that saves an id other than the recorded one names the previous id's `--adopt` command in its hint, after the watcher-stopping hint when both apply; `--continue` and `--adopt` of the recorded id emit none (D1).
- `lyx orch status` reports `session_id` (D6).
- The `start` and `status` help, `internal/orchcli/doc.go` and `internal/orchengine/doc.go` describe the flag, the record and the reboot recovery, as the Scope's last In bullet asks.

No recorded decision is broken: `--continue` stays an explicit flag and a plain `start` still relaunches from the last handoff (D1), and orch parses no transcript or provider path (D5, `PATTERN-shuttle-provider-seam`).
The Out items, reed's rebirth resume path and `session-sync`, are untouched; the rebirth residual is recorded in `internal/orchengine/doc.go`.

## Carried-over risks

The Plan-Review carry-over `continue-bound-omits-provider-reliance` is answered in the change: D3's bound names the reliance on the provider naming the parent session id in a subagent's or fork's hook lines, and `internal/orchengine/doc.go`'s Open risks gains that entry.
The decision record's other open risks are provider behaviours the design accepts, and the change does not widen them.

## Overlaps

- `creel-stage1-fixes` (row 8, own row 22): both change `internal/shuttleengine`.
  This run adds a session id field to the files-only session reading and an accessor on `Run`; the peer changes the input-box capture, the idle probe's stall reasons and a creel interrupt variant.
  The changes sit at different places with no shared behaviour, decision or PATTERN entry, so it is a text-only overlap.
  Landing order: this run, at row 22, lands first, and `creel-stage1-fixes` takes its changes at Merge-In.
- The parent lists no landed commits.
- `cmd/lyx/testdata/test-budget.yaml` raises `internal/orchengine` from 144 to 145, a shared bookkeeping file only.

## Superseded

No other board entry is made obsolete by this run.
