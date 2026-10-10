# batten run --window opens a terminal window on the task worktree

`lyx batten run <slug> --window` now also opens one desktop terminal window for the run, titled with the slug and running `lyx reed attach` in the task worktree.
The orch no longer opens that window by hand with a Konsole command line.

## What changes

- **`internal/termwindow`** (new) picks a terminal launcher per platform and opens a detached window with it.
  On WSL (detected by `WSL_DISTRO_NAME` or the `WSLInterop` binfmt entry) it runs `wt.exe -w new --title <slug> wsl.exe --cd <dir> -- bash -lc …`, escaping `;` for Windows Terminal's own command-line splitting.
  On other Linux machines with a graphical session it runs `konsole --workdir <dir> -p tabtitle=<slug> -e bash -lc …`.
  Anywhere else, `Resolve` returns an error wrapping `ErrNoLauncher` that says why.
  The window's shell line runs the spawning `lyx` binary's `reed attach`, then `exec bash`, so the window stays open after a detach.
- **`battencli`**: under `--window` the run verb resolves the launcher before it opens the tmux window.
  When there is a launcher, the run started in the tmux window gets the hidden `--open-terminal` flag, and the envelope's `terminal_window` names the launcher.
  When there is none, the run goes on without the flag, and `terminal_window` says no window was opened and why.
  An already-live tmux window reports no `terminal_window`, because nothing is started.
  `wire` fills `InnerRun.OpenTerminal` only when `--open-terminal` is set.
  It locates the task worktree, resolves the launcher and opens the window at the worktree's anchor.
- **`battenshed`**: `InnerRunDeps.OpenTerminal` brings back the once-per-run open that the VS Code removal took out, now pointed at a terminal window.
  It runs after a successful spawn, both on the first spawn and on an approved or rejected resume.
  A once-marker (`<producer>-terminal-opened`) in the row's scratch directory gates it, and a fresh child clears the marker.
  A failed open is a warning only, and the marker is written anyway.
  Re-running `lyx batten run`, or re-spawning the child, therefore never opens a second window.
- Docs: CLAUDE.md "Watching runs" drops the hand-written Konsole line.
  The orch role stencil tells the orch that batten opens the window.
  The `battenshed` package doc, `docs/overview.md` and the run verb's help describe the new behavior.

## Tests

- `termwindow`: launcher choice across WSL, X11, Wayland, no session, no `konsole` and non-Linux, plus the exact Konsole and Windows Terminal argument vectors.
- `battenshed`: three tests restored from the removed IDE-open tests.
  They pin the order (spawn, then open, then read), the once-marker across resumes and stale markers, no open after a failed spawn, and the warning-only failure.
- `battencli`: `TestRunInWindow` covers the forwarded `--open-terminal` and both forms of `terminal_window`.
  The absent-worktree seam test covers `OpenTerminal`, which is nil without the flag.

The real launch was not exercised, so the WSL path is untested on an actual WSL machine.
Windows Terminal's quoting of the forwarded `bash -lc` line is the part most likely to need adjustment there.
