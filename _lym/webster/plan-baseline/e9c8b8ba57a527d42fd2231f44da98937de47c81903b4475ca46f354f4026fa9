# Card 7 — kwinplace-prelaunch

**Create:**
- `internal/kwinplace#PreLaunch`
- `internal/kwinplace#OpenWindow`
- `internal/kwinplace#TestPreLaunch`

**Extends:**
- `internal/kwinplace/doc.go`

**Uses:**
- `internal/termwindow#Launcher.IsKonsole`
- `internal/termwindow#Launcher.Open`

**Intent:** Add D1's pre-launch checks and the one window opener both window paths call (D1, D2, D12).

`PreLaunch` reads the environment of the calling process, through an unexported probe of `getenv` and `lookPath` like termwindow's,
and returns nil when every check passes:
`KDE_SESSION_VERSION` is `6`, `XDG_CURRENT_DESKTOP` names `KDE` among its colon-separated entries, `XDG_SESSION_TYPE` is `wayland`,
`konsole` is true, and `qdbus6`, `journalctl` and `kscreen-doctor` are on `PATH`.
Otherwise it returns an error naming the first check that failed.
The names of the variables, values and tools are named constants.

`OpenWindow` opens a terminal window on `dir` through `launcher`, with no branch of its own:
it takes `placed` as `PreLaunch(launcher.IsKonsole()) == nil`, logs at Info that placement is skipped and why when it is not,
once per window (D1's bound), and returns `launcher.Open(dir, title, lymPath, placed)`'s pid with `placed`.
`Open` with `placed` false is today's command line, and no KWin call is made on that path.
The choice is pinned by `TestPreLaunch`, and the command line each choice runs by card 5's `TestLauncher_OpenRunsTheChosenCommandLine`.
`doc.go` gains a paragraph on the two.

`TestPreLaunch` is `tier0`, rows over the probe: every check passing; `KDE_SESSION_VERSION` unset and `5`;
`XDG_CURRENT_DESKTOP` `GNOME`, and `KDE` inside a list such as `KDE:custom`, which passes; `XDG_SESSION_TYPE` `x11`;
a launcher other than Konsole; and each of the three tools missing from `PATH`.
Each failing row asserts the error names its check.
It adds under 0.1 seconds to one `tier0` run.

**ImpactSummary:** Two new exported functions in the new package; nothing calls them until cards 13 and 14.
