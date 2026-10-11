# Support log — pane-inject-safety

## Interview

Autonomous mode: no operator answered; every turn below is a self-pick made after reading the board entry and the code it names.

1. Read the board entry (two merged faults, operator 2026-10-11) and the related entries `creel`, `shuttle-color-verified-send`, `orch-owns-the-loop`, `parent-review-hardening`, `run-message-routing`, `startup-probe-wait` and `reed-tmux-conf`.
   `creel` fixes the future layering (`creel → shuttle → reed → tmux`), so the design puts keystroke mechanics in reed and the safety decision in shuttle (D1).
2. Read the send path: the pre-typing idle wait, the Escape plus 300 ms settle, `send-keys -l` typing, settle reads, the unchecked first Enter, the 48-character needle match, and the session commands that type on the caller's probe.
   Confirmed every fault the board lists.
3. Scope question: is a persistent queue needed for "waits in a queue and is retried"?
   Every caller already has one (orch notice files and cycle phases, the gate's unsent re-prompt, seat notices' busy retry, reed's per-tick name repair); only the orch watcher counts a busy send as a tick failure and only the loom resume path drops it.
   Pick: fix those two callers, build no queue (D7, D8, D9).
4. Atomicity versus the last guard: board item 2 puts the Enter inside the atomic tmux list, board item 3 wants a box check right before the Enter.
   Pick: the check before the atomic list is the pre-write guard (D3); every Enter lyx sends on its own afterwards is exact-match guarded (D5).
   This also makes the live-check outcome safe either way: an Enter turned into a newline normalizes to an exact match and gets one guarded Enter.
5. Where the operator-activity window lives: reed owns tmux client facts and both shuttle and reed's name repair need the number, so it is one `reed.yaml` key (D3).
6. Session rename: reed's watchdog types it without shuttle and cannot import shuttle; routed through reed's own paste-submit and activity query (D6).
7. Batten: retry versus `blocked`. Pick retry plus one orch notice per stall (D9), status flag for any halted batten over a done child (D10), and the orch stencil step (D11).
8. PATTERN: the guarded path is a new cross-cutting invariant, so it gets an entry (D12).

## Rejected alternatives

- **Check the box after the paste and before a separate Enter, as the primary sequence.** The paste needs a redraw settle before the capture shows it, and keys the operator presses in that window merge into the box; refusing the Enter then leaves the draft spoiled, which the board calls too late.
  Kept only as the fallback variant if the live check shows Claude Code swallows an Enter after a paste end.
- **A tmux `if-shell -F` condition inside the command list to test activity atomically.** A command-line client has no client context for `client_activity`, and pane content is not a format; the millisecond gap left is accepted as R2.
- **A shuttle- or reed-owned persistent injection queue.** Duplicates the callers' existing queues and the planned `creel` queue.
- **Batten halts `blocked` with the remedy "clear the draft, then `lyx batten run <slug>`".** Needs a human for a transient condition and reproduces the stranded-teardown failure if nobody resumes.
- **Fail closed when the operator-activity query errors.** A tmux that cannot answer `list-clients` would stall every injection on the hub; the empty-box and Enter guards still protect the draft.
- **Clear the box with C-u on any mismatch so the send can retry.** The board forbids editing the operator's box; clearing stays limited to an exact match of this send's own text.
- **Move name repair into shuttle by changing the `SessionNamer` seam to a rename call.** The per-hub watchdog covers every worktree, while a shuttle `Runner` is bound to one worktree's run records; reed's own primitives reach the same guards with less coupling.
- **Surface sent text left in the operator's draft as a non-retried error naming the pane.** Every retrying caller (the watcher, batten, seat notices) would stop at it and need a human; the pending-send record in D5 lets the existing retries run without typing the text twice.
- **Key the pending-send record per strand, whatever the text.** A different text sent next (a second notice) would be dropped as landed when the first one's turn started; fixing the resume line's report path per park (D8) gives the one caller whose text changed between attempts the same text instead.
- **Stop the watcher and seat notices from retrying a not-landed send.** Both would need a human for a menu the operator closes on their own; the pending-send record covers the outcome for every turn-starting text, and a repeated slash command is harmless (R8).
- **Hold the activity check over the whole submit window before a placeholder Enter.** The window is `submit_confirm_timeout_s` long, so an operator's keystroke anywhere in it would hold the Enter until the send fails; the collapse-length and line-count match narrow the acceptance instead, and the residual is R3.
- **Have shuttle's `Start` wait or retry indefinitely on a held delivery.** `Start` would have to hand back a half-started run or block its caller without a bound; a died start after `send_ready_timeout_s` keeps today's contract.
- **A 3 s quiet window.** Ordinary thinking pauses mid-sentence exceed it; 5 s costs little on injections that already wait for idleness.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Build a persistent queue? No; fix the two callers that drop a held write (D7, D8).
- Q2 (resolved, auto-pick): Atomic paste+Enter or paste, check, Enter? Atomic paste+Enter after a pre-write guard, every later Enter exact-match guarded (D2, D3, D5).
- Q3 (resolved, auto-pick): Quiet window default and owner? 5 s, `reed.yaml` key `operator_quiet_s` (D3).
- Q4 (resolved, auto-pick): Activity query error? Fail open with a Warn (D3).
- Q5 (resolved, auto-pick): Unreadable box? Held, for box-reading engines only (D4).
- Q6 (resolved, auto-pick): Drop the leading Escape everywhere? Yes; prompt suggestion is off in shuttle-launched sessions (D6).
- Q7 (resolved, auto-pick): Batten retry or blocked? Retry plus one notice per stall (D9).
- Q8 (resolved, auto-pick): Flag only `failed`? No; failed, blocked and paused (D10).
- Q9 (resolved, auto-pick): Route rename through shuttle? No; through reed's primitives (D6).
- Q10 (resolved, auto-pick): `Runner.Interrupt`, `Runner.Inject`, `ModelSwitchSequence` in scope? No; explicit key deliveries that bypass idleness on purpose, and `ModelSwitchSequence` has no production caller.
- Q11 (open, for the plan): `set-buffer` or `load-buffer` to fill the named buffer (D2).
- Q12 (open, for the operator): run the `llm`-tier live check; agents in this run may not run `llm`-tier tests, so D5's fallback covers the result until then.
- Q13 (resolved, auto-pick): Sent text left beside the operator's text in the box? Its own sentinel wrapping `ErrSessionBusy`, plus a pending-send record that dedupes the retry by a turn start past the send's offset (D5); a not-landed send after a no-box window records one too.
- Q14 (resolved, auto-pick): A held `Start` delivery? Stays a died start after `send_ready_timeout_s`; a foreign-box failure dies at once (D7, R7).
- Q16 (resolved, auto-pick): How does the pending-send record match a resume line whose report path is fresh on every attempt? The report path is fixed once per park and reused on every rerun (D8).
- Q17 (resolved, auto-pick): Does the probe's pass clear a send hold in `State.Stuck`? No; a send hold has its own marker, cleared only by a passing send (D7).
- Q15 (resolved, auto-pick): Does the batten orch notice quote the draft? No; slug, stall and remedy only (D9).
