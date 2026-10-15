# Support log — reed-bindings-repin

## Interview

A darn run holds no interview: the board entry and the code are the whole input.

## Rejected alternatives

- **Read an unset `@lyx_follow` as off.** The board entry's scope keeps follow's default as it is, and the stale bindings would still lack any later binding change.
- **Re-pin on every op unconditionally.** It costs a socket-path read and every `bind-key` on each op, and a long-lived older process would pin its stale bindings back over a newer build's.
- **Stamp a hash of the binding argvs instead of the build.** It re-pins on any content change, but carries no order, so an older and a newer binary running ops side by side would alternate the bindings without end.
- **Re-pin in each verb's own engine method.** Every verb would carry the same call; `withOpLock` is the one site they share.
- **Re-pin the global status-bar options too.** They are pinned interleaved with session-scoped options in `pinGeometryOptionsLocked`, and the board entry names the bindings; left as an open risk.

## Review rounds

_No rounds yet._

## Question ledger

- **Which reed ops re-pin?** Self-pick: every op under the blocking op lock; the watchdog's try-lock re-apply and name repair stay off it, since they run on a tick.
- **What identifies a build?** Self-pick: `buildvcs.Identity`, its commit time ordering builds and its revision label telling them apart.
- **What does an unstamped binary do?** Self-pick: it never re-pins a running server, and its own pin unsets the stamp so the next stamped build re-pins.
- **Do older builds re-pin?** Self-pick: no; only a build whose commit time is not before the stamped one re-pins.
