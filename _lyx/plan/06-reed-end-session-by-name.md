# Card 6 — reed-end-session-by-name

**Create:**
- `internal/reedengine#EndSessionByName`
- `internal/reedengine/endsession_test.go`

**Uses:**
- `internal/reedengine#ReapSession`
- `internal/reedengine#Engine.Down`

**Intent:**
Gives the pair-teardown composite (card 9) a way to end a pair's session when the pair's worktree is already gone.
`Engine.Down` cannot do this, because `withOpLock`'s live-worktree validator refuses a gone root, and creating the op lock would recreate `<hub>/<slug>/.lyx`.

`EndSessionByName` is the third engine-less exported function in `overlay.go`, beside `ReapSession`, and keeps its rules: no lock, no state directory, and no geometry field beyond the two strings it is told.
It runs these steps in order:
1. Asks tmux whether the exact-match session exists (`hasSession`).
   An absent session returns `(false, nil)` and touches nothing.
2. Reaps it through `ReapSession`, which computes the descendant closure before the kill.
3. Tidies the server the way `Down` does when no sessions remain: an empty or failed `list-sessions` leads to `kill-server`, and the socket goes with the server.

It logs the reap at Info, as `ReapSession` already does (Live-Substrate Spawn Observability).
The doc comment states why it exists beside `Down` and that it must never be pointed at a session whose worktree is still present, where `Down` is the right call.

Bound: it skips the op lock and the state write only because both lived under the gone worktree, so neither has anything left to guard.
It reaches only the exact-match session it is named, on the socket it is given, and kills that socket's server only when no session remains on it.
Its one caller is `internal/pairteardown`, which calls it only when the pair's task worktree path is not a registered linked worktree (card 9); every other session end goes through `Down`.

The body sits in an unexported variant that takes a `TmuxCmd`, the way `reapSessionPanes` and `reapSessionKill` split out of `ReapSession`, so a test can drive it through `TmuxCmd`'s exec seam.

Tests in `endsession_test.go` (untagged) drive that seam with the package's fake tmux (`faketmux_test.go`):
- An absent session issues no kill and no kill-server.
- A present session with siblings remaining issues the exact-match kill-session and no kill-server.
- A present last session issues both.

**Commit:** `6: reed ends a session by name for a gone worktree`
