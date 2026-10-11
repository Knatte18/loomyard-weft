# reed: opt-in tmux_conf key to source an operator tmux config

Since the hygiene change, reed's per-hub tmux servers start with `-f /dev/null` and read no `~/.tmux.conf`, so the operator lost their prefix, key bindings and `escape-time`.
This adds a `reed.yaml` key, `tmux_conf`, that names a tmux config file reed sources into a hub server it has just started.
Empty, the default, leaves servers hermetic exactly as today.

## What changes

- `reed.yaml` (both templates) gains `tmux_conf: ${env:LYX_REED_TMUX_CONF:-}`, with a matching `Config` field.
  `lyx config reconcile` adds it to an existing file.
- On POSIX, a non-empty value is validated on every boot: a leading `~/` is expanded, and the result must be an absolute path to an existing regular file with no whitespace and none of `*?[\'";#{}$`.
  Anything else is refused like an unresolvable `shell`, naming `tmux_conf`, `LYX_REED_TMUX_CONF` and the path.
- The boot that starts the hub server runs `source-file <path>` after its session is up and before reed's option and geometry pins, so reed's pins win over the file.
  The capability probe and the server spawn keep `-f /dev/null`.
- Windows parses the key but neither validates nor sources it; the Windows template comment says so.
- `tmuxkit.Main` unsets `LYX_REED_TMUX_CONF`, so no test server sources a developer's config.
- Docs: the template comments, the "Operator config" paragraph of `internal/reedengine/doc.go`, and the "Hermetic servers" section of `pattern/PATTERN-test-isolation.md`.

## Design decisions

- **One sourcing boot.** `serverSpawnArgv` runs whenever a worktree's session is missing, so it does not identify the starter.
  A boot reads the server's `#{pid}` and `#{start_time}` before and after its spawn and is a candidate only when they differ.
  A candidate claims the server with an `if-shell -F` check-and-set on the server user option `@lyx_tmux_conf_claimed`, which tmux runs without interleaving other clients, so exactly one of several concurrent boots sources the file.
  Nothing re-sources into a running server; a changed value takes effect at the server's next start, like `debug_log` and `mouse`.
  In a hub, the worktree whose boot wins the claim decides the file.
- **Sourcing errors warn.**
  A parse or run-time error in the file logs a Warn record and prints the warning to stderr, and the boot continues.
  The guard invocation carries the claim, the source and the pins under one exit status, so its status never decides a pin failure.
- **Reed guards its session and server against the file.**
  The spawn chains a session-local `destroy-unattached off` and a `@lyx_reed_session` tag after `new-session`.
  The guard invocation then sets global `destroy-unattached off` and server `exit-unattached off`, and a separate invocation reads both back and checks the session still exists.
  A file that ends the boot's session, or leaves either option on, fails the boot with a message naming `tmux_conf`, `LYX_REED_TMUX_CONF` and the path.
  The pin set is closed at these two options; any other option the file flips is the operator's to own.
- **Sessions the file creates do not hold the server.**
  On a server whose claim reads `sourced`, a session counts as reed's only if it carries `@lyx_reed_session`.
  Every reed read that lists, counts or cycles sessions, in Go or in a tmux format such as the status bar's session row, sees only those, so the last worktree's `down` still ends the server.
  On any other server, including one started before this change, every session counts as before.
- An operator file with a waiting command (`run-shell` without `-b`, as tpm's last line) lets other clients run between the file's lines and reed's pins; the chained session-local guard and the non-waiting claim keep that window harmless.
- A starter whose boot ends between its spawn and its guard invocation leaves the server unclaimed and file-less until its next start; no recovery path is built.

## Check by hand

- No test covers the case where a boot's pre-spawn read saw a server that has ended by its spawn: that boot should be a candidate and source the file into the server its spawn starts.
  Check the before/after comparison in `internal/reedengine/lifecycle.go` for it.
- The status-bar integration test is constrained to `tmux && linux` rather than `tmux && !windows`, since tmux kills its server on `#{S:}` expansion without an attached client and the PTY harness that attaches one is Linux-only.
- Not run: a real tmux config with plugins (tpm, status scripts) against a live hub.
  Such a file adds background CPU to every hub server, and other options' `run-shell` and `#()` jobs still run.
- An untargeted session-level command in the file applies to whichever session tmux takes as current, which under a concurrent boot can be another worktree's session.
  The chained `destroy-unattached off` covers every reed session; nothing else does.
