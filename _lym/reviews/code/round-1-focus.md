---
round: 1
focus:
  - "internal/machinecli/start.go: does a prime owned by another machine, or with no recorded owner, start only the beams owned here, name the prime's machine, and never claim the prime or go through `lym s`?"
  - "internal/machinecli/start.go: does each started worktree get a terminal running `lym reed attach`, and does the verb print each attach command where no launcher resolves?"
  - "internal/machinecli/start.go: does a standing hold print its machine and subjects, plus the `lym adopt <subjects>` re-run for this machine's own hold only?"
  - "internal/machinecli/start.go: is `lym beam start` run as a child process named through `selfexec.LymBinary` and `internal/runproc`, with the spawn logged, per PATTERN-child-processes?"
  - "internal/dobbyengine/launcher.go, catchup.go, loop.go and internal/battencli/catchup.go: is the launcher added at the dobby start for a worktree owned here, a beam under another machine's prime included, and removed by every catch-up that finds nothing owned here, the dobby start's included? Is it kept while anything is owned, and does `internal/beamcli` stay out of it?"
  - "internal/machinecli/start.go and internal/desktoplauncher: does the verb remove each launcher on disk whose hub `hubregistry.Live` no longer lists, and the all-hubs launcher with the last hub, while the all-hubs launcher stays as long as any hub has one?"
  - "internal/desktoplauncher/locate*.go and desktoplauncher.go: are the Linux `.desktop` file, the Windows `.cmd` file and the WSL `wsl.exe` route correct, including directory choice, quoting and escaping, and does the code build for both platforms?"
  - "internal/termwindow: does native Windows (`wt.exe`) resolve and build its arguments correctly, and do the existing Linux and macOS behaviours stay unchanged?"
  - "Repository rules: do the new refusals name a way forward and have a test reaching it, does `internal/dobbyengine`'s seam allowlist hold, and do docs/overview.md and the package doc.go files cover the diff?"
  - "Test economy: the report lists `TestCatchUp_RemovesTheDesktopLauncherOnceThisMachineOwnsNothing` as contained, covered by `TestRun_StartAddsTheDesktopLauncherOfAWorktreeOwnedHere`. Compare their assertions before filing a fold, delete or keep finding."
---

Seed focus drawn from the decision record's D10 and its acceptance criteria, checked against the diff stat of the review range.
