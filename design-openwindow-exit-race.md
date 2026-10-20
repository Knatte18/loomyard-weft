# reedengine: TestOpenWindow_RealTmux/ClosesWhenItsCommandExitsAtOnce flakes: window read "on"

`openwindow-exit-race` · Note · bug, reed

Imported from [issue #761](https://github.com/Knatte18/loomyard/issues/761).

## Seen (2026-10-20, OSL-1046: Ubuntu 24.04, tmux 3.4, Cortex XDR)

The darn run `arbiter-verdict-quoting` failed its Publish verify only on `internal/reedengine` `TestOpenWindow_RealTmux/ClosesWhenItsCommandExitsAtOnce`, with `window @3 read "on"`.
The branch does not touch `internal/reedengine`, and the full verify (both tiers) had passed on the same head `43c40f29` minutes earlier at Darn-Write.
It looks like a race in `OpenWindow`'s `remain-on-exit` read-back when the window's command exits at once.

On this machine Cortex XDR holds every newly built binary 8-10 s before it runs, so timing differs a lot from other machines; that may widen the race.

Batten sent the first Publish failure back to Darn-Write as designed; the darn writer cannot run the tmux tier (see board note `darn-verify-tier-unrunnable`), made no change and handed back for a Publish retry.

## Expected

The test is deterministic: `OpenWindow` and its test do not depend on whether the command has already exited when `remain-on-exit` is read back.