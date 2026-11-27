---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Neither proposed option is taken. A wait under a minute is too short for a start-every-hub burst, and a 1 s to 8 s poll adds up to 8 s of idle lock per waiter. D10 takes the lock through lock.AcquireWriteLock, a blocking flock that wakes only when the lock frees, as the Slack holder's lock wait does (PATTERN-daemon-wakeups: 'its lock wait blocking without waking'). A timer ends it at 120 s and the child exits. Q14 restated; D10 acceptance criterion added."
  - id: F2
    disposition: fixed-differently
    reason: "The proposed rule, wait until the parent's entry is placed, still races: the self-check found that an entry settled from the last session ends the wait at once, then the parent's new pre-wait write lands before the child's snapshot. D8 now reads the anchor under the lock: a settled entry gives the parent's live desktop, an unsettled one gives its home desktop, where its own placement sends it. The child waits only when there is no entry or the entry has no desktop. The D8 criterion is changed as proposed, and the case of a child taking the lock before its parent's child is added."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: deferred
    reason: "Each observation needs a KWin script loaded through qdbus6 on the operator's live session bus, and building a grid on a desktop moves the operator's real windows. This session may not reach the user session bus and must route such work to the operator. The in-surface parts are done: the desktop and output move added to D3's apply step, the probe's mechanism replaced, and the unobserved facts listed in Constraints with the operator named as who runs them."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: "D10's window wait runs read-only D3 queries outside the lock, a second apart, as proposed. D8's parent wait does not read the same signal: after F2 the anchor comes from the parent's places entry, which only the places file holds, so that wait re-reads the file and needs no script."
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed-differently
    reason: "The D3 unload-on-every-exit change and the timeout criterion are made as proposed. The D10 'build new tiles before releasing old ones' change is not made, because a desktop cannot be left half-built: once run is called, the script's body runs whole inside KWin whatever happens to the caller. D3 and D10 now say so. A SIGTERM unloads the script and settles the entry, and a SIGKILL leaving an inert loaded script is a bound. The fact is listed among Constraints' unobserved facts."
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
---

# Round 1 fixer-report — discussion

## Changes

All edits are in `_lym/discussion/decision-record.md` unless the support log is named.

- F1 (fixed differently, see frontmatter): D10 takes `kwinplace.lock` through `lock.AcquireWriteLock`, a blocking acquire with a 120 s timer after which the child exits. Rationale cites `PATTERN-daemon-wakeups`. D10 acceptance criterion added. Support log: Q14 restated, and polling the lock added as a rejected alternative.
- F2 (fixed differently): D8 reads the anchor under the lock from the parent's entry. A settled entry gives the parent's live D2 desktop or the entry's last desktop. An unsettled entry gives its home, else its last desktop. The child waits outside the lock only when there is no entry or the entry holds no desktop. D9 adds the settling process id and the settled state, and both D8 criteria are rewritten.
- F3: D12 states the real order (`runBeamStarts`, then `openWindows`) and drops "parents first". D8's wait is 180 s, re-reading from one second and doubling to eight, and is named for `PATTERN-daemon-wakeups`'s list. The first-start fallback is a D8 bound. The D12 criterion about parents' windows opening first is removed. Support log: rejected alternative added.
- F4: D6 clamps the bottom row inside the apply script through `frameGeometry` and `clientArea`, so no `kdotool` is needed. It states the 1-row case and gives a clamped window its zone by frame position. D3's apply step names the clamp. The Notes line on `arrange-to-tiles.sh` is corrected, and a D6 criterion added.
- F5: D9 makes every write one `state.UpdateJSON` read-modify-write that changes only its own fields, and the pre-wait write keeps home, order key and last desktop. D9 criterion added.
- F6: D9 keeps a home desktop and order key apart from the last desktop, an overflow keeps the home, and the overflow bound is stated. D8 reads the home. D9 criterion added.
- F7 (deferred): D3's apply step sets the placed window's desktops and calls `workspace.sendClientToScreen`. Constraints lists the unobserved KWin facts with the operator as who runs them. The probe now compares the root tiles of two desktops instead of checking arity.
- F8: D2's rationale records the observation and justifies `--separate` by Konsole's single-process setting. `pgrep -a konsole` was run by this fixer on 2026-11-27. The `kdotool getwindowpid`/`getwindowname` result, pid 3913 with caption `ORCH [ly:P0] root — Konsole`, is the review's own run: kdotool goes through the session bus, which this session may not reach. Support log: the caption rejected alternative is corrected.
- F9: D1 splits the pre-launch checks, which decide the command line, from the post-launch probe, whose failure leaves the window unplaced and settles its entry. The D1 and D2 criteria are updated.
- F10 (fixed differently): D10's window wait runs read-only queries outside the lock, each run starting at least a second after the last ends. D8's wait reads the places file. D10 criterion added.
- F11: D10 names `kwinplace.lock` in `standalonestate.UserStateDir`, where the machine record of hubs and `launchers.lock` sit.
- F12 (fixed differently): D3 unloads by plugin name on every exit, a SIGTERM unloads, and a script runs whole in KWin. A SIGKILL is a D3 bound. D10 says how a child ended mid-apply leaves the desktop and its entry. D3 criterion added.
- F13: D12 makes the child run `window_command` on every exit path, mode `off` included. `lym machine start` gains it under placement only and runs none when the pre-launch checks fail. Two D12 criteria replace the old one.
- F14: a second Review criterion covers `lym reed window`'s Long help and the template's `window_command` and `window_wait_ms` comments.
- F15: D9's renumber puts present windows into the positions present windows held and keeps absent entries' positions, so no keys collide. D9 criterion added.
- F16: D3's rationale drops the second-granularity reason.
- F17: D6 keeps "exactly" and rewords the rationale so it states the departure from `arrange-to-tiles.sh`.
- Support log: a Round 1 entry under Review rounds, and Q16 to Q19 added to the ledger.

## Self-check

- fix-consistency, MEDIUM, a failed probe leaves the window "unrecorded" after the pre-wait write: fixed. Every child exit without an apply settles its entry (D1, D9, D10), and the D1 criterion is updated.
- fix-consistency, LOW, "remembered desktop" ambiguous after the home/last split: fixed. D8's fallback chain names home and last desktop.
- fix-consistency, LOW, D9's "only its own entry" omits the drop: fixed. The `UpdateJSON` sentence names the drop.
- fix-way-forward, MEDIUM, failed probe "unrecorded", and other unplaced exits stall children: fixed, merged with the item above. Settling on every exit covers the lock and window timeouts too.
- fix-way-forward, MEDIUM, the unobserved-facts step names no actor: fixed. Constraints names the operator.
- fix-way-forward, LOW, a blocking acquire has no way to end at 120 s: fixed. D10 says the child exits with the acquire pending.
- fix-claims: no findings.
- fix-proposal, BLOCKING, the hand-move rule rewrites the home of an unsettled reopening window: fixed. Unsettled entries are exempt (D9), and a D9 criterion added.
- fix-proposal, BLOCKING, the race in which the parent's entry becomes unsettled after the child's wait ends: fixed. The anchor is read under the lock, and an unsettled entry gives its home (D8).
- fix-proposal, MEDIUM, failed probe "unrecorded": fixed, merged with the fix-consistency item.
- fix-proposal, MEDIUM, the clamp unties every window of a 1-row desktop under a bottom panel: fixed. D6 states the case, and a D6 criterion added.
- fix-proposal, LOW, the D8 bound misses recorded-but-unplaced parents: fixed. Settling on every exit leaves only a parent whose first child was killed outright, now in the bound.

## Disputed

None.

## Deferred

- F7: observing the KWin behaviour on Neve needs KWin scripts run over the operator's user session bus, and a grid build moves the operator's real windows. This session may not reach the session bus and is told to say so to its parent. The operator, or a run the operator approves, must run each fact listed under Constraints' "Not yet observed" bullet and record its command and output there before code relies on it.

## Rulings

None.
