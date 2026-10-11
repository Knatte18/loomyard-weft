# Round 2 fixer report — discussion

Every finding was validated against reed's code and a scratch tmux 3.6 server before fixing, and all five held.

## Fixed

- **F1 (BLOCKING)** — confirmed: three runs of `source-file` with `set -g exit-unattached on` followed by a separate `set -s exit-unattached off` each got "server exited unexpectedly"; the pin chained into the same invocation kept the server up.
  D5 step 2 now sends one guard invocation per boot, the candidate's claim followed by `-g destroy-unattached off` and `-s exit-unattached off`, with step 3's session check in a separate invocation after it.
  The false rationale sentence is replaced by the one-client-from-source-to-pin reason.
  Support log: new rejected alternative and updated Q8.
- **F2 (MEDIUM)** — confirmed: after the file's `set -g destroy-unattached on`, a plain `new-session -d` died at once and one with `set destroy-unattached off` chained survived.
  D5 step 1 now chains the session-local `destroy-unattached off` and D8's `@lyx_reed_session` tag into the spawn invocation; D3 states the spawn argv gains that guard while keeping `-f /dev/null`; D8 states the tag exists from the session's creation, closing the concurrent-`down` window.
  New acceptance criterion: two concurrent boots with a file setting both `-g destroy-unattached on` and `-g exit-unattached on` both succeed.
  The third open risk now says what the chained guard covers.
  Support log: new rejected alternative.
- **F3 (MEDIUM)** — confirmed against `Down` (`lifecycle.go`, kill-server on an empty `list-sessions`) and the per-worktree op lock.
  D3 step 1 now compares the server's `#{pid}` and `#{start_time}` before and after the spawn, with the rationale for both values and for pre-change servers.
  New acceptance criterion for a seen server that ends before the spawn.
  Support log: the pre-spawn-probe rejected alternative gains this case; Q12 updated.
- **F4 (LOW)** — confirmed: an unquoted path with a space inside the `if-shell -F` branch was split into two missing files; a quoted one loaded.
  Chose refusal: D2 now also refuses whitespace and `'`, `"`, `;`, `#`, `{`, `}`, `$`, with the rationale; the path acceptance criterion gains a path with a space.
  Support log: rejected quoting alternative; Q13 updated.
- **F5 (LOW)** — confirmed: `SwitchClient` (`switch.go`) cycles every `list-sessions` id and `statusFormatSessions` (`bar.go`) loops `#{S:}` over every session.
  D8 now states one rule over every reed read that lists, counts or cycles sessions, in Go or a tmux format, with a widened bound; the file-created-session acceptance criterion covers the bar row and the switch.
  Support log: rejected list-of-surfaces alternative; Q11 updated.

## Disputed

None.

## Deferred

None.
