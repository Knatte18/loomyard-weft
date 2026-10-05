# Support log — orch-cycle-hardening

## Interview

Autonomous session: no operator answered.
The board entry carries three operator decisions from 2026-10-05 (cycle mode default `compact`, `permission_mode: prompt` removed, `orch-no-watcher` in scope); every other choice is an auto-pick recorded in the ledger.

1. Read the board entry and the related entries `orch-owns-the-loop`, `parent-review-hardening`, `batten-windows`, `stencil-loads-skills`.
   `orch-owns-the-loop`'s "watch command" overlaps `orch-no-watcher`; this task covers it, and the overlap is listed as an open risk for the board.
2. Read `internal/orchengine/doc.go`, `watcher.go`, `template.yaml`, the orch stencils and `claudeengine/usage.go`.
   Confirmed the incident's cause: the reading moves only at a turn end, and the reader takes the last assistant entry.
   `tickIdle` fires only with a turn end as the newest event, so a re-read before firing always has a transcript reference.
3. Whether background shells survive `/clear`: the orchengine doc's Open risks records the 2026-10-02 smoke run, in which a background task's completion reached the resumed session after `/clear`.
   So cycles never required re-arming; only a new process (adopt, reboot) kills them.
4. Read `internal/reedengine/windowsize.go` and `doc.go`: `window-size latest` is a recorded, load-bearing reed decision (the attach chain is suppressed on any other value).
   The attach item is therefore handled on the orch side (D16).
5. Read `claudeengine/resume.go`: the registry check refuses on any live pid, which is the PID-reuse false refusal.
6. Read `internal/battenshed/doc.go` and `internal/battencli/wire.go`: the Run-Shed row polls the child status and already has a `DriverAlive` seam; the notice writer fits beside it.
   No Go path to SendMessage exists, so a notice must be typed into the orch pane, which only the orch watcher does safely.

## Rejected alternatives

- **Reading: only refresh on a compaction hook event.** `/compact` produced no event in the incident's event file, and auto-compaction mid-turn has no separate event either; the transcript boundary plus a re-read covers both.
- **Reading: re-read on every tick.** Costs a transcript read every poll; a re-read only at firing time is enough to stop a stale fire.
- **Compact cycle with a DEFER handshake** (a question turn, then `/compact` on a READY reply). Adds a phase and a round trip for a risk compaction does not carry.
- **Compact cycle followed by a resume prompt.** The session was idle before the cycle and keeps its summary; a prompt would only spend a turn.
- **Compact cycle that still writes a handoff file.** The operator decided the instruction goes into `/compact`'s focus and no file is written.
- **Removing the `permission_mode` key outright.** The board decision says the key and its comment state that bypass is the only mode; removal would also break every existing `orch.yaml` under strict loading.
- **Batten types notices into the orch pane directly through reed.** Two Go writers would race each other's idle checks and could type into a turn the other just started.
- **A `lyx watch` command the orch runs in a background shell.** It is still a watcher that dies at adopt and reboot, which is what `orch-no-watcher` removes.
- **Parked-driver detection by probing the driver pane for a prompt.** Provider-specific and brittle; the quiet-status notice covers parked and slow alike.
- **Changing reed's `window-size` to `largest` or `manual` for the prime.** `internal/reedengine/doc.go` records why each breaks the attach chain or client following.
- **PID reuse: ignore the registry when the pid looks reused by name.** Process names are not a reliable identity; the recorded start time is.
- **Large-session adopt: a fabricated padded transcript.** Depends on guessing Claude's transcript chaining; copying a real transcript tests the real shape.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, operator): cycle mode config and default → `cycle_mode: clear|compact`, default `compact`, focus argument carries the handoff instruction, no handoff file.
- Q2 (resolved, operator): `permission_mode: prompt` → removed; `bypass` only.
- Q3 (resolved, operator): watchers → `orch-no-watcher` in scope; gaps closed in lyx code.
- Q4 (resolved, codebase): do background shells survive `/clear`? → yes, per the 2026-10-02 smoke run recorded in `internal/orchengine/doc.go`.
- Q5 (auto-pick): how a compaction resets the reading → latest of boundary and assistant entry by file order, plus a re-read before a hard or soft fire (D1, D2). Covers manual and automatic compaction with no new event source.
- Q6 (auto-pick): how the watcher detects a finished compaction → a boundary timestamped at or after the phase entry plus an idle probe (D6). The only signal `/compact` leaves.
- Q7 (auto-pick): DEFER in compact mode → none (D7). Compaction does not lose in-flight work the way `/clear` does.
- Q8 (auto-pick): repeated failing compactions → `compaction timed out` holds automatic triggers for `soft_idle_s` (D8). Prevents a retry loop at every idle grace.
- Q9 (auto-pick): what `permission_mode: prompt` does now → config load error naming the fix (D10). Silent coercion would hide a stale config.
- Q10 (auto-pick): how batten reaches the orch → a notice queue the orch watcher delivers idle-gated (D13), with a new PATTERN entry. Keeps one Go writer into the orch pane.
- Q11 (auto-pick): which gaps batten notices cover → state change off running, dead driver strand, quiet status for `notice_quiet_min` (default 45) (D12). Matches the three cases `battenshed/doc.go` excludes.
- Q12 (auto-pick): when there is no orch → batten logs only (D13). Matches the board's "parent is optional".
- Q13 (auto-pick): the orch pane's height → keep reed's `window-size latest`; report a too-short pane in `lyx orch status` (D16). reed's doc records why the window policy stays.
- Q14 (auto-pick): reading after adopt → unknown until the first turn end (D4).
- Q15 (auto-pick): PID reuse → compare the registry's recorded start with the live process; fail closed when unreadable (D15).
- Q16 (auto-pick): large-session adopt → opt-in smoke test over a copied transcript, run once, result recorded in doc.go (D18).
- Q17 (open, for the board): `orch-owns-the-loop`'s "A watch command" section is superseded by D12–D13 and needs amending; this session may not write the board.
