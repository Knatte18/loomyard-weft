# reedengine: TestOpenWindow_RealTmux/ClosesWhenItsCommandExitsAtOnce flakes: window read "on"

`openwindow-exit-race` · Task · bug, reed · running · Darn-Burler-Fix

Imported from [issue #761](https://github.com/Knatte18/loomyard/issues/761).

## Seen (2026-10-20, OSL-1046: Ubuntu 24.04, tmux 3.4, Cortex XDR)

The darn run `arbiter-verdict-quoting` failed its Publish verify only on `internal/reedengine` `TestOpenWindow_RealTmux/ClosesWhenItsCommandExitsAtOnce`, with `window @3 read "on"`.
The branch does not touch `internal/reedengine`.
Darn-Write's verify had passed on the same head `43c40f29`, but it runs only the unit and integration tiers, so Publish was the run's first tmux-tier run.
It looks like a race in `OpenWindow`'s `remain-on-exit` read-back when the window's command exits at once.

On this machine Cortex XDR holds every newly built binary 8-10 s before it runs, so timing differs a lot from other machines; that may widen the race.

Batten sent the first Publish failure back to Darn-Write as designed; the darn writer cannot run the tmux tier (see board note `darn-verify-tier-unrunnable`), made no change and handed back for a Publish retry.

## Hypothesis (orch, 2026-10-20, from reading `internal/reedengine/window.go`, not verified)

`OpenWindow` chains `new-window ... ; set-option -w remain-on-exit off ; ...` and then reads the value back with `display-message -p -t <window id> '#{remain-on-exit}'`.
When the command exits at once, the window may already be gone by the read-back.
The code treats a failed read as "gone", but tmux may not fail: it can resolve a target it cannot find to the current window and print that window's value, `on` from the session-wide option, which `OpenWindow` then reads as a mismatch.
The sibling subtest whose command runs for a second passes, which fits.
So the window most likely closed as intended, and only the read-back is wrong.

A fix could read `#{window_id} #{remain-on-exit}` and count a reply whose id is not the new window's as gone, and the fix checks how tmux 3.4 and 3.5 resolve a vanished `-t` target.

## Expected

The test is deterministic: `OpenWindow` and its test do not depend on whether the command has already exited when `remain-on-exit` is read back.