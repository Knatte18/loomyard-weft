Task: check the sets behind F1, F2, F10 and F12 for members the fix missed.

Findings: none.

Sets I checked, one search each:
- **F12:** `freedesktopDesktopDir` has two callers, `locate.go:51` and `locate_test.go:111`. Both pass the new parameters.
- **F1:** `claimPrimeNotes` is reached only from `prepare` (`orchcli/start.go`), which `beamUp` calls with `!opts.NoPrimeClaim`. `beamcli.start` is called only from `startCmd`. The `BeamUpOptions` callers (`adoptcli`, `loomcli`, `orchcli.BeamUpWorktree`) leave `Prepare` false, so they never reached the claim.
- **F10:** The only `wsl.exe` command-line builder in a `.cmd` is `desktoplauncher.go:158`, which the fix covers. `termwindow` builds an argument vector instead, which is not the double-quoted `bash -lc` form F10 describes.
- **F2:** `docs/overview.md:867` describes the verb at a level that stays true, and `machinecli/doc.go` was updated with the new `worktrees` and `line` fields.

Out of scope, noted only: `fabriccli/anchordeps.go:339` also runs `lym beam start <label>` as a child. It is not part of the F1 review, and I did not check whether it claims a prime.

Calls made:
1. Bash grep of `freedesktopDesktopDir(` callers, `beam start` spawners, `claimPrime*` sites, `machine start` docs and `wsl.exe` sites. My context lacked the whole-repo callers.
2. Bash `sed` of `docs/overview.md:865-885`. My context lacked the doc text.
