---
round: 2
focus:
  - "D2 in decision-record.md drops the explicit logger discard in the watchdog verb (internal/reedcli/watchdog.go, logger.SetOutput(io.Discard)); SpawnWatchdog (internal/reedengine/spawnwatchdog.go) uses exec.Command with a nil Env, so a detached daemon inherits LYX_LOG_FILE and LYX_LOG_LEVEL, which internal/logger reads at init. Settle whether D2 states what a detached daemon now does with an inherited log file and what bounds it, and whether an acceptance criterion pins that."
  - "D4 places the zero-pin rebuild in pinGeometryOptionsLocked, which the boot path (internal/reedengine/lifecycle.go) and AttachArgv's pre-flight (internal/reedengine/attach.go) both call; the attach path returns before installResizePinsLocked on an in-closure degrade (see internal/reedengine/doc.go on attach). Settle whether D4 names the call site unambiguously and states what a degrading attach with the watchdog on does to the session's resize pins."
  - "D7 launches the per-hub tmux server with -f /dev/null and pins default-shell and default-command per session. Settle whether D7 states what the server now ignores of the operator's tmux settings and what bounds it, and whether any way forward it names (including the Open risks line) exists as a real surface; reed.yaml templates are under internal/reedengine/template_posix.yaml."
  - "D7 pins the shell options at a fresh boot and in the attach pre-flight but resolves the shell with exec.LookPath in the boot path's pre-tmux block. Settle where the attach pre-flight gets the resolved shell and what it does when the shell does not resolve, and whether the Decision states that the options are per session and never set with -g because one server serves every worktree of the hub."
  - "D5 bounds POSIX poll mode as transient after D4, while internal/reedengine/watchloop.go can stay in poll mode for good after a failed file watch or a failed hook install, and parks on ctx.Done() under watchdog: off. Settle whether D5's stated bound matches those cases."
  - "D8 attributes the benchmark's children CPU to the watchers' poll-mode round trips. Settle whether D8 presents that as established or as a hypothesis, and whether it decides what the last card records when the re-measurement does not confirm it."
---

Focus entries target the open design questions of the decision record.
Each can be settled by reading `_lyx/discussion/decision-record.md` against the cited code.
