---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/reedengine/tmuxconf.go:33
    summary: A ~/ value with an empty home directory expands to a root-anchored path and passes the absolute check instead of being refused
  - id: F2
    severity: LOW
    class: scope
    location: internal/reedengine/tmuxconf_integration_test.go:107
    summary: The refused-boot subtest does not assert that no reed session or server is left behind, and the session-gone refusal leaves a server alive when the file created a session
  - id: F3
    severity: NIT
    class: consistency
    location: pattern/PATTERN-test-isolation.md:53
    summary: The new Hermetic servers bullet is separated from its sibling bullets by a blank line and sits above the bullet that introduces the pre-started server it contrasts with
---

The range is `git diff $(git merge-base main HEAD)..HEAD`: no rework round exists and `product.parent` is `main`.
The tmux-tier package run `lyx gate test --tags tmux ./internal/reedengine` passed.
No `Rename` or `Delete` group appears in the four cards, so the per-card mechanical checks have nothing to run beyond the build and the tests.
Every `list-sessions` argv outside the `listSessionsArgv` definition goes through it (a grep over non-test Go shows only the definition and the allowed-verb list in `probe.go`), and `statusFormatSessions` carries the filter in both loop bodies.

Focus items settled with no finding:

- Card 3, sourcing once: the claim is an `if-shell -F` check-and-set on a server option, run in `guardBootSessionLocked` after the session is up and before `pinBootOptionsLocked`; candidacy comes from comparing the identity read before the spawn with the one after, so only a boot that saw a different server (or none) enters the claim, and the claim itself picks one winner.
- Card 3, target form: the two chained `set-option -t` calls in `serverSpawnArgv` use `exactSessionWindowTarget`; the guard's `-g` and `-s` pins take no target.
- Card 3, read-back: the guard's exit status only logs and writes a warning; `hasSession` and the chained `show-options` read-back decide, and `TestTmuxConfBootGuard` pins both failures.
- Card 2: `preflightBootHost` validates on every POSIX boot, after `resolveShellPath`, with an error in the `key shell` error's style.
- Card 4: all session reads and the status-bar format apply the filter, and `-f` is passed off Windows only.
- Card 1: the key is parsed on both platforms, validated on POSIX only, and `setEnv` unsets the variable.
- Test economy: the new tests each pin a behavior the others do not; the `listSessionsArgv` rows and the `wantArgv` literals are what the cards require, and the other call sites are covered by their scripted-fake tests.

### [LOW] An empty home directory turns `~/x` into `/x`

**Location:** `internal/reedengine/tmuxconf.go:33`, with `internal/reedengine/lifecycle.go` (`preflightBootHost`).
**Issue:** `preflightBootHost` passes the empty string as `home` when `os.UserHomeDir` fails, intending the `~/` path to then fail the absolute check.
`tmuxConfPath` computes `home + path[1:]`, so `~/tmux.conf` becomes `/tmux.conf`, which is absolute.
The value is then looked up at the filesystem root, and either sources an unrelated file or refuses with an error naming a path the operator never wrote.
`TestTmuxConfPath` has no row for an empty `home`.
**Fix:** Refuse a `~/` value when `home` is empty, naming the original `raw` value, and add a row for it.

### [LOW] A refused boot is not shown to leave nothing behind

**Location:** `internal/reedengine/tmuxconf_integration_test.go:107` (`UntargetedDestroyUnattachedFailsTheBoot`), `internal/reedengine/tmuxconf.go` (`guardBootSessionLocked`).
**Issue:** The subtest asserts only that `Up` returns an error naming the key, the variable and the path.
It does not assert that the server is gone afterwards.
In that scenario the server ends because its only session died, but a file that also creates a session (`new-session -d -s operator`) before ending reed's leaves a live server holding only a session reed hides, and the refusal path does not tear it down.
**Fix:** Assert in the subtest that `ListSessions` reports no server after the refusal.
If the file-created-session case is meant to leave the server alive, state that in `doc.go`'s bound; otherwise end the server on the session-gone refusal when no reed session remains.

### [NIT] Bullet placement in `PATTERN-test-isolation.md`

**Location:** `pattern/PATTERN-test-isolation.md:53`.
**Issue:** The "Hermetic servers" section's bullets have no blank lines between them, and the new bullet adds one after itself.
It also precedes the bullet describing the pre-started server it refers to as the contrast.
**Fix:** Place the bullet after the `Socket` and `KillOnCleanup` bullet with no blank line between bullets.
