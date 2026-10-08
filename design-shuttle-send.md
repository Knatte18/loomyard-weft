# Shuttle: a Send lands or retries, and attach knows a half it stopped

`shuttle-send` · Task · bug, shuttle · running · Publish

Depends on: [`shuttle-state`](design-shuttle-state.md)

A text shuttle sends into an agent pane lands, and when it cannot, the run retries instead of halting.

Seen 2026-10-08 in `plan-glyph-precision`, under high machine load: the skill-load `Send` into a freshly started Burler half left the text pending in Claude Code's input box after 2 extra Enters, the step failed as `producer-hard-failure`, and the driver had to re-step by hand; the re-step then halted on a run dir it could not classify.

## From issue #442

## What happened

A loom run halted `failed` (anomaly `producer-hard-failure`) at the `Discussion-Burler` row, round 2 attempt 2.
The error chain:

```
shedadapters: Discussion-Burler (burler): round 2 attempt 2: run: burler: shuttle run: shuttle: load skills [scribe:prose scribe:code-quality scribe:testing]: shuttle: Send: the sent text is still pending in the input box after 2 extra Enter(s); the submission did not land
```

Earlier rows in the same run (`Discussion-Write`, and the first `Discussion-Burler` attempt) completed; this was the first submission failure.

## Why it is abnormal

The shuttle's skill-load `Send` types the text into the agent's tmux pane, presses Enter, and gives up after 2 extra Enters when the text is still in the input box.
A transient pane state (agent busy, input not yet ready, paste not yet consumed) turns into a hard producer failure that halts the whole run.
Nothing in the failure distinguishes "agent not ready to accept input" from "submission lost", and no retry or wait-for-idle precedes the give-up.

## What to do

- Before giving up in `Send`, wait for the pane to be ready for input, then retry the submission with backoff rather than a fixed 2 extra Enters.
- Treat a failed skill-load submission as a retryable step (re-send once, or restart the shuttle attempt) instead of `producer-hard-failure`.
- Record the pane's last lines in the halt note so the cause (busy agent, modal prompt, unconsumed paste) is visible.

Provenance: task `plan-glyph-precision`, from `loom-halt.md`.



## From issue #443

**What happened**
During a loom run, the Discussion-Burler step (burler round 2, attempt 2) failed with a non-transient `producer` error from `shuttle: Send`:
`the sent text is still pending in the input box after 2 extra Enter(s); the submission did not land`.
The step was refused (`shed: step refused kind=producer`), so the skill load into the freshly started half never happened.
The trace had just logged `burler: stopping a live half of a round` before the refusal.
The only `fabric: mutation` records were a records commit and its push, so no half-finished add/remove was left behind.

**Why it is abnormal**
A Send into a live agent pane should either submit or fail with a retryable condition.
Giving up after 2 extra Enters turns a timing/pane-readiness problem into a hard, non-transient producer failure that halts the run as `failed`.

**What to do**
Make `shuttle: Send` verify pane readiness before sending (or retry with backoff and re-check the input box) and classify the pending-input outcome as transient so the shed retries the step instead of halting.
The driver recovered by re-stepping (`interrupt_policy: reinvoke`).

Provenance: task `plan-glyph-precision`; notes: `driver-repair-20261008-discussion-burler-1.md`.



## From issue #444

**What happened**
Re-stepping Discussion-Burler after the Send failure (see the companion issue on `shuttle: Send` leaving text pending) halted the run as `failed` with anomaly `producer-hard-failure`:
`round 2: probe the round's halves: burler: probe the burler-review half: shuttle: attach: 1 candidate run dir(s) under .lyx/shuttle cannot be confirmed dead or alive (untracked or with a cleared pane binding, younger than 3m0s): .lyx/shuttle/<run-id> — check "lyx reed status" and either wait or clear the stale directory by hand`.
The run history at the halt had 6 entries (Discussion-Bouncer/Burler alternating `stuck`).

**Why it is abnormal**
The candidate run dir is almost certainly the half that `burler: stopping a live half of a round` had just stopped in the previous attempt.
lyx created that state itself, yet the next probe cannot classify it and treats it as a hard failure instead of waiting out the 3m window.
An automated run should not need a manual "wait or clear by hand".

**What to do**
Have the stop path mark the run dir dead (or clear its pane binding explicitly) so the probe sees it as dead.
Alternatively make the probe's "cannot be confirmed" outcome transient, so the shed retries after the grace window rather than halting.

Provenance: task `plan-glyph-precision`; notes: `loom-halt.md`, `driver-repair-20261008-discussion-burler-1.md`.



## Direction

- Find the root cause in `Send` first: when it types, how it detects pending input, and when it presses Enter (paste not yet consumed, a session still starting, a readiness probe passing too early).
- Wait for the pane to be ready before typing and for the typed text to settle before Enter; retry Enter with backoff within a bounded time, re-reading the input box.
- A submission that still does not land is a transient error, so the step retries; the error carries the pane's last lines.
- `shuttle attach` tells a half that lyx itself just stopped from an unknown live run, so a re-step never halts on it.


## No send into a running turn after a restart

From `parent-review-hardening` (PR #336 review, 2026-10-02): `AttachGated` replays events from offset 0, so the first tick treats the writer as idle even mid-turn: a notify-carried prompt or a reject re-prompt can be typed into a running turn, and a consumed reject is let through while the writer is still fixing.
A send waits until the session is idle in fact, not by a replayed reading.
