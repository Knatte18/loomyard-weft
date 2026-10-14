# tender: hub-wide pause, resume, revive and status

`lyx pause` and `lyx resume` were two top-level commands from `internal/pausecli`, so `lyx --help` listed them as if each were a module.
They now live in a `tender` module with the verbs `pause`, `resume`, `revive` and `status`; `lyx pause` and `lyx resume` stay as shortcuts that forward the slug and flags.
The change also stops a pause from eating the deadline of a GitHub call it holds, and makes the long-lived processes that can restart safely pick up a replaced binary.

Closes #606

## What changes

- `internal/pausecli` is renamed `internal/tendercli` and `internal/pauseengine` is renamed `internal/tenderengine`; `internal/pauserecord` keeps its name.
  `pause` and `resume` behave as before.
- `lyx tender status` replaces `lyx pause --status`, which is now an unknown flag.
  It also lists every live agent session in the hub (pair, strand, state, cause, whether a live pause entry covers it, the orch included) and the lyx processes still running a replaced binary.
  The stale listing reads `/proc` on Linux, through a seam implemented beside `internal/proc`, and reports itself unsupported elsewhere.
- `lyx tender revive [<slug>]` types one fixed carry-on line into each session stalled idle on an `api-error`.
  Paused sessions get no line, since `resume` gives it.
  The orch gets a queued `orchengine` notice instead of typed text, and the watcher drops the notice at delivery if the orch has recovered.
- `internal/shuttleengine` gains a conditioned send for `revive`.
  It refuses at once, with no idle wait, on any mismatch of state or cause, a draft, operator activity, an unmatched turn start, an open rewind picker or a stray character.
  It never clears its own text, and after landing it waits up to one submit window for the turn start.
- A GitHub call held by a pause now gets its full deadline after release (#606).
  `pauserecord.Network` reports the held time, and `githubclient` drops the http client's `Timeout` and applies its 30-second timeout itself in the transport, moving the deadline later by the held time.
  A GitHub deadline error stays transient, so the loom drive rules re-step it once before the run halts `failed`.
- `lyx orch watch` re-execs itself on a replaced binary at a tick with no cycle step in flight and no live hub pause entry.
- `lyx reed attach` on Unix execs the tmux client in its own place, so no lyx process lingers.
  It must be the last argv of any shortcut, and the shortcut guard in `cmd/lyx` now checks that.
- The docs, `pattern/PATTERN-cli-cobra.md`, the test budget rows and `tools/sandbox/SANDBOX-CORE-SUITE.md` follow the rename and the new verbs.

## Design decisions to review

- `revive` reads a session's pause coverage and runs its conditioned send, the turn-start wait included, inside `pauserecord.Update` with a mutate that changes no entry.
  This holds the pause record's write lock, so a concurrent pause takes its entry only after the revived turn's start reads or the wait ends, and `pauserecord` gains no call and keeps its import set.
  The cost is that a concurrent `pause`, `resume` or watcher record write waits at most one ready reading plus two submit windows per session.
- Two races stay unrecorded by design: a send that lands with no turn start seen within the wait, and a revived turn that ends before the pause's poll.
  `revive` reports the first as sent with no turn start, and `status` shows the session busy under the live entry, so the operator can pause that slug again.
- Whether the watcher re-exec can fire while a pause entry is live was settled as never, handled or not, so no in-memory pause handling is lost across the exec.
- The reed attach exec skips the root's post-run, so the logger is flushed and the exec logged first.

## Check by hand

- Review the findings the design and plan reviews left open:
  - the conditioned send's lock hold across `revive`'s sends is bounded by windows, not by a hung reed or tmux call, which each wait on their own untimed operation lock or tmux call;
  - the `revive` and `pause` race described above, in `internal/tenderengine`;
  - `revive` sends no key of its own beyond its typed line and submit, which the conditioned send in `internal/shuttleengine` should keep;
  - the held-page row's 30 ms slack in the `githubclient` pagination test;
  - the paste row in the `revive` tests, whose API-error parser may be unreachable.
- Run `lyx tender status` and `lyx tender revive` in a live hub, which no automated test covers end to end.
- Replace the `lyx` binary under a running `lyx orch watch` and confirm it re-execs and leaves one watcher.
- Run `lyx reed attach` and confirm no lyx process remains once tmux is attached.
