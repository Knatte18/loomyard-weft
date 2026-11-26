```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the reviewer's proposals that the fixer carried out as written.

Proposals challenged: F1, F2 (supervisor-socket half), F3, F4, F5, F6, F7, F8, F9, F10, F11, F12, F13, F14, F16, F19, F20, F21, F22, F23.

1. **MEDIUM**: F10's read grant exposes other hubs' deploy clones and relay setup.
   - Changed text: D6's read list says "every hub's area state directory (D3)". D8 says "A daemon reads every area hub's supervisor record from its area state directory, which D6 grants read inside every area."
   - Summary: the grant covers each whole area state directory. Those directories also hold D13's `lym area deploy` clone and D12's relay setup, so every area's agents can read every other hub's cloned repository and Slack setup. That breaks the Goal's "read a secret outside it".
   - Evidence: D13 says `lym area deploy` "fetches the parent branch from origin into its own clone in the area state directory". D12 says "The relay's setup, the operator id and the channel and bridge maps, lives in the area state directory".
   - A sound change grants read on each hub's supervisor record file alone, not the directory.

2. **MEDIUM**: F1's "watchdog starts only inside the area" names no spawn site, so an area hub gets no watchdog.
   - Changed text: D6's attach bullet says it "skips `EnsureSession`, `Status` and the watchdog spawn; … and the watchdog starts only inside the area".
   - Summary: the host-side attach is the only place that spawns the watchdog today. Once it skips the spawn, nothing in the record starts the watchdog for an area hub.
   - Evidence: `internal/reedcli/spawnwatchdog.go:12` is the only production caller of `reedengine.SpawnWatchdog`, reached from `internal/reedcli/attach.go:85`.
   - The record needs to name the site that starts it, such as the supervisor's `boot` or dobby, or state that an area hub runs without one.

3. **MEDIUM**: F5's broadened refusal contradicts the cross-hub creel that D6/D8 require.
   - Changed text: D7's first sentence says "every `lym` verb run outside the area refuses over that hub, whether it resolves the hub from its cwd, the machine record or a shortname". Its exempt list does not include creel.
   - Summary: a host-hub `lym creel post` to an area hub, and the host daemon's delivery, resolve the area hub by shortname from the machine record. The first sentence refuses that, yet the D6, D8 criterion requires delivery and confirmation in both directions.
   - Evidence: the later D7 sentence "Cross-hub creel reads an area hub's machine-record entry alone" implies creel is allowed, but the exempt list ("bar `lym area`, `lym reed attach`, the supervisor-started relay of D12, `lym version` and help") does not name it.
   - Cross-hub creel `deliver` and `confirm` belong in the exempt list.

4. **MEDIUM**: F9's "carries a seccomp filter" clause changes behaviour for hubs without an area.
   - Changed text: D8 says a daemon serves only `deliver`/`confirm` to a peer "that carries a seccomp filter and is not inside the daemon's own hub's area". The same bullet does this for a peer "whose process is gone or unreadable".
   - Summary: every daemon, including those of hubs with no area, now refuses in-hub posts from any filtered process. Examples are a command run under Claude Code's own bash sandbox (bwrap plus seccomp), or a tool run from a sandboxed terminal. The Goal says "With the area off, everything works as it does today."
   - Evidence: host processes carry filters routinely. On Neve, `chrome` shows `Seccomp_filters: 1`, as the fixer's own `/proc` scan printed.
   - A narrower rule keys the refusal on the area's recorded attach clients, which the fixer already added to D3's inside test, rather than on any filter.

5. **LOW**: F1's attach order leaves no way to detect "no server answering" before it restricts itself.
   - Changed text: D6 says attach "skips `EnsureSession`, `Status` … with no server answering it asks the supervisor's `boot` verb … and only then runs tmux, its floor check included".
   - Summary: the server check has to happen before attach restricts itself, and the record names no tmux-free way to do it, such as a plain connect to the derived socket path.
   - Evidence: the same bullet forbids running tmux before the restriction.

Further tool calls:
- `internal/reedcli/attach.go` lines 80–90 and the `ensureWatchdogSpawned` definition, to confirm where the watchdog is spawned; my context showed only grep hits for it.
- The `SpawnWatchdog` doc comment and the hibernate packages' use of the machine record, to check whether another spawn site exists and how hibernate reaches hubs; neither was in my context.
- All production callers of `SpawnWatchdog`, plus the hibernate package listing, to confirm attach is the only spawn site; my context lacked the call graph.
