```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guarantee and claim about existing code in the decision record and tried to break each with a concrete input or interleaving.

**Claims attacked**
- D1/Constraints, "D1's 8 KiB claim budget keeps each field under the [10,000-character] cap": **broken** (F1).
- D1, the claim, hold, pause and acknowledge-after-write order "stay": **weakened** under D2's merged command (F7).
- D2, "no loop, since each block claims at least one message": **held**. A block is printed only when the claim is non-empty.
- D2, "the count errs early, never late": **held within one daemon life**, **broken across a daemon restart** (F2).
- D2, "Staying below Claude Code's cap of 8 means no override ever discards a block whose mail creel journaled read": **broken** by a daemon restart mid-chain (F2).
- D2/D4, the count resets on a `PostToolUse` claim or a `Stop` with `stop_hook_active` false: **broken in practice**. The reset only reaches the daemon while the flag stands (F3).
- D2 rationale, "`ParseEvents` reads every `Stop` line as `EventStop` or `EventWaiting`": **held** (`internal/shuttleengine/claudeengine/events.go:45-57`).
- D3 bound, one `lym` process per subagent tool call while mail stands: **held**. The flag test gates the spawn (`settings.go:124`).
- D4, 86400 s is under the JS timer ceiling: **held** (86,400,000 ms < 2,147,483,647 ms).
- D4, the waiter reconnects up to `MaxReconnects` and never exits 2 on a drop: **held** (`internal/creelengine/client.go:20,157`).
- D4, "a ring that races a turn the operator starts delivers nothing": **held**. The waiter claims nothing.
- D4, the idle check "through the `Delivery` seam reports no running turn": **broken as a claim about existing code** (F5).
- D4/D5, "once per claimable set": **held within one daemon life**. A restart repeats the ring as well as the bell, and D4's bound names neither (part of F2).
- D5 bound, "only into a shuttle-launched strand": **held**, with a marker that exists (`Provider`). The bell's text misleads pre-deploy shuttle-launched sessions (F4).
- D6 rationale, "every hub session is launched by shuttle with the hooks": **broken** by D5's own hand-added strands and by pre-`Provider` records (F6).
- D7, "a post that fails validation sends no key" and "a refused interrupt stores nothing": **held**. A store failure after the key is sent is uncovered (F8).
- D7 bound, it stops a turn only where `interrupt` already may: **held**, since it uses the same sender rules.
- D8, one line shape: **held**.
- D9/Constraints, "both `shell.Quote` forms carry `"` intact" to the launch argument: **held at the shell-string level**, **unverified one hop past** (F9).

**Findings**

**F1 — BLOCKING, design.** Location: Constraints (JSON output cap), D1, D6, Acceptance criterion "A claim at the 8 KiB budget arrives whole".
The 8 KiB budget does not keep a field under 10,000 characters, because `ClaimAtBoundary` always hands out the first message whatever its size.
A body may be up to 64 KiB.
- Counterexample: a 20 KiB post becomes the one claimed message. `reason` and `systemMessage` each exceed 10,000 characters, so Claude Code replaces them with a preview. hook-read wrote the output, so creel acks it as read. The tail of the message is lost silently.
- This contradicts the entry's and D6's "delivered in full".
- Today a program notice over 4 KiB falls back to the reminder and `lym creel read`. D6 removes that fallback.
- Evidence:
  - `internal/creelengine/verbs.go:162` ("always hands out the first")
  - `internal/creelengine/verbs.go:188-194`
  - `internal/creelengine/pool.go:17` (`maxBodyBytes = 64 << 10`)
  - `internal/creelengine/reminder.go:23,28-30`

**F2 — MEDIUM, design.** Location: D2 Bound and rationale, D4/D5 "once per claimable set".
D2 does not say the continuation count and the per-set wake memory survive a daemon restart, and the daemon keeps per-strand state in memory.
- Counterexample: the strand is 5 blocks into a chain, so Claude Code's count is 5 and creel's is 5. The daemon restarts (a deploy; dobby now survives deploys, commit `1ca6c020`) and creel's count drops to 0. Creel then allows up to 6 more blocks. Claude Code overrides the 8th consecutive block, and that block's mail was already journaled read: the loss D2 says cannot happen.
- The same restart re-rings a set D4 already woke, which costs one extra wake turn. D5's bound names the repeated bell, but D4's bound does not name the repeated ring.
- Evidence: `decision-record.md` D2 ("Staying below Claude Code's cap of 8…"), D5 Bound ("a restarted daemon may type it once more"). The journal fold carries only message records (`internal/creelengine/fold.go` is listed; no count record is named).

**F3 — MEDIUM, design.** Location: D2 count reset, D4 "rings no more … until the count resets", D5's at-cap trigger.
The flag test gates the creel step, and `hook-read` is the only thing that passes `stop_hook_active` to the daemon. So a `Stop` with `stop_hook_active` false and no mail standing never resets the count, and a turn whose tool calls claim nothing never resets it either.
- Counterexample: the count reaches 5 in turn A, and turn A's last block takes all the mail. In turn B, the operator's turn sees no mail, so there is no reset. While the strand is idle, a post arrives. D4 rings, and the count becomes 6. The woken turn's `Stop` carries `stop_hook_active` true (the record's own rejected alternative), so the count is at cap and claims nothing: one wasted wake turn. D5's bell is then typed into the pane.
- Once the count is at 6 it stays there across any number of mail-free turns. Every later idle wake for that strand becomes a typed bell instead of a ring until a `PostToolUse` claim happens.
- The record also leaves open whether a `stop_hook_active`-false `Stop` resets the count before or after the cap check.
- Evidence:
  - `decision-record.md` D2 ("The flag test gates only the creel step"; "hook-read passes stop_hook_active")
  - `support-log.md` Rejected alternatives ("a rewake turn carries it as true")
  - `internal/shuttleengine/claudeengine/settings.go:124` (flag test, then `exit 0`)

**F4 — MEDIUM, design.** Location: D5 and its bell text (D4), Open risks line 3.
A session launched before the deploy has `Provider` set, so D5 counts it as shuttle-launched and bells it. Its `Stop` hook is still the plain recording command and has no delivery step.
- The bell tells it: "End this turn now; creel hands the mail to you as the turn ends. Do not run lym creel read for it."
- Counterexample: an obedient pre-deploy agent ends the turn with no tool call. Nothing delivers, and D5 never repeats the bell for the unchanged set. The agent was told not to use the one route that works for it.
- Open risks says only that D5 "may bell them once", not that the bell misdirects them.
- Evidence:
  - `internal/reedengine/state.go:41-43` (`Provider` is set by shuttle's spawner)
  - `internal/shuttleengine/open.go:88` and `internal/shuttleengine/run.go:600`
  - `internal/shuttleengine/claudeengine/settings.go:314` (old `Stop` is recording only)
  - D4's bell text

**F5 — LOW, design.** Location: D4, "the strand's idle check through the `Delivery` seam reports no running turn, where a draft in the input box does not count as busy".
The seam provides no "running turn" reading.
- `Idleness.Stall` is empty both for a running turn and for an active operator. It is set only for too-short, draft and no-input-box.
- D4 therefore cannot tell a running turn from an active operator. It also does not say whether too-short or no-input-box (a dialog open) holds the ring.
- Evidence: `internal/creelengine/seam.go` (`Idleness` comment: "empty … on any not-idle reading that names no reason, such as a running turn or an active operator"; the `Stall*` constants).

**F6 — LOW, design.** Location: D6 rationale and Bound.
"Every hub session is launched by shuttle with the hooks" is false by D5's own text: a hand-added strand (`lym reed add`) exists and has no hooks.
- After D6, such a strand gets no notice, no reminder and no bell, so agent mail to it sits unread with no signal at all.
- A strand recorded before the `Provider` field decodes with none and is likewise excluded.
- D6's Bound names neither case.
- Evidence: `internal/reedengine/state.go:41-43` ("added outside shuttle, decodes with none"), `decision-record.md` D5 last sentence.

**F7 — LOW, design.** Location: D1, "acknowledge-after-write order stay", together with D2's merged command.
In the merged `Stop` command, hook-read writes into the shell (a variable or capture), not to Claude Code. The ack therefore follows a write that is not yet delivered.
- Counterexample: the shell fails between the capture and the print, or Claude Code's hook timeout kills the command. The mail is journaled read and never shown.
- Evidence: `decision-record.md` D2 ("when that prints a block, the command prints it"), `internal/creelcli/hookread.go` (`writeContext`, then the client acks).

**F8 — LOW, design.** Location: D7.
The order is validate, then interrupt, then store. A store that fails after the key was sent, such as a journal append error or a strand removed in between, leaves an interrupted turn with no post. D7 names no outcome for that case.
- Evidence: `decision-record.md` D7, `internal/creelengine/verbs.go:140` (journal append can fail).

**F9 — LOW, design (deferred research question).** Location: Constraints (`internal/shell`), D9.
The pwsh `Quote` single-quotes, so `"` survives the shell string. The constraint stops there, one hop short of the native argument.
- Under Windows PowerShell or pwsh before 7.3, and for a `.cmd` shim under 7.3's `Windows` mode, embedded `"` in a native-command argument is stripped or re-split.
- The record does not establish how `claude` is resolved on Windows, so whether D9's sentence arrives verbatim there is unverified.
- Evidence: `internal/shell/pwsh.go:12-18` (`Quote`, `& 'bin'`).

**Tool calls**
1. `grep` of `internal/shell` `Quote` and of hand-added markers: the context lacked the `Quote` forms behind D9's constraint (returned nothing usable).
2. `grep` of `internal/shell/*.go` `Quote` plus the reed strand types: the context lacked the pwsh `Quote` and where the strand record lives.
3. `sed` of `internal/reedengine/state.go` `Strand` plus `Provider` in creel: needed the marker D5 would use to tell shuttle-launched strands from hand-added ones.
4. `grep` of who adds strands and sets `Provider`: confirmed shuttle's launch paths set it (`open.go:88`, `run.go:600`) for F4 and F6.
5. (Calls 1 to 4 are all the calls made.)
