```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked every claim the fix added or rewrote against the code at HEAD.

Claims checked:
- **Holds.** `hook-read` acknowledges only after its write, and a killed hook leaves its claim to come again. The read flag is cleared only when the read is journaled (`internal/creelengine/doc.go:105`), and a dropped connection releases the reservation (`server.go:439-465`).
- **Holds.** `forward` keeps the line it forwards. `Forward` only appends `eventForwarded` and wakes the receiver, and it changes neither `From` nor `ID` (`verbs.go:321-345`, `41-46`).
- **Holds.** The 2.1.312 claims are right: per-turn guards start at `stopHookBlockingCount:0`, a tool continuation spreads `...pi`, the rewake enqueues `stopHookActive:!0`, and the cap is `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP??8`. `--append-system-prompt-file` exists.
- **Holds.** pwsh 7 is required by `OrElse`'s `||`, and `claude.yaml`'s `binary` is the configured binary.
- **Fails.** "a blocked chain never gives" a no-running-turn reading, and the hook-derived readiness reading decides `NoTurn`. See finding 1.

Findings:

1. **MEDIUM.** A ring can reset creel's count in the middle of a woken turn's blocked chain, so the count can run late.
   - **Changed text:** D2 Bound, "a ring resets creel's only on D4's reading of no running turn, which a blocked chain never gives, since its blocked stops record no turn end". Also D4, "the hook-derived readiness reading decides it".
   - **Why it fails:** the hook-derived reading only reports a turn as running when a recorded turn start has no turn end yet. A wake turn records no `UserPromptSubmit`, as D4's own Bound says. So during a woken turn the reading stays on the prior turn's `Stop` record and reads idle-done. `judgeReadiness` (`internal/shuttleengine/readiness.go`) holds a reading only for `SessionBusy` with a running turn, an unmatched turn start, a draft or a bad box.
   - **What goes wrong:**
     - Every blocked `Stop` also starts a D4 waiter, because the `asyncRewake` entry runs on each `Stop`.
     - Mail that arrives mid-chain in a woken turn finds a live waiter and `NoTurn` true, so it gets a ring.
     - The ring resets creel's count to 0 while Claude Code's count is k > 0.
     - Creel can then allow up to 6 more blocks, Claude Code overrides the 9th, and mail creel journaled read is discarded. That is the loss D2's "errs early, never late" bound rules out.
   - **Second problem:** this also contradicts D4's Bound, which says "shuttle's guarded send reads the pane as busy during it". That is not what the hook-derived reading `NoTurn` uses gives.

Further tool calls:
1. Grepped creelengine for when the read flag is cleared and for `deliverMailLocked`/`Forward`'s effect on `From`, and checked the `claude.yaml` name. The context did not show flag timing or the forward path.
2. Grepped for where the readiness reading is built, to test the `NoTurn` and "blocked chain" claims. The context did not show readiness code.
3. Read `internal/shuttleengine/readiness.go:17-120` (`judgeReadiness`), to see which session states hold a reading. The context lacked it.
