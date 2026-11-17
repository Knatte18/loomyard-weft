---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed per-worktree line and hub label would not reach the terminal summary, because output.Summary prints a list item's one line and its notes only, never a nested list (internal/output/summary.go:161-186). The worktrees moved to a top-level worktrees list of the start result, each labelled <shortname>/<worktree> with a line; the hub keeps the name label and carries the hold as its line. The hold object stays beside it for machine readers."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed-differently
    reason: "The lock path is <per-user state dir>/launchers.lock, from standalonestate.UserStateDir, rather than a path taken from internal/lock, whose helpers build tracked-tree paths only. Add holds the lock across its two writes, and Remove across the hub removal, the Shortnames read and the all-hubs removal; a lock failure is an error the callers already log as a warning. No new test: the finding is a design fix, and a race test would need a real-time interleaving."
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
---

# Fixer report, round 1

## What changed

Commits, one per target; a finding that shares a file with others is committed with them and named in the message.

- `beamcli: fix F1`: `lym beam start` takes a hidden `--no-prime-claim` flag.
  `startOptions` carries it into `orchcli.BeamUpOptions.NoPrimeClaim`, and `prepare` skips `claimPrimeNotes` when it is set.
  `machinecli` appends the flag to every by-slug child; the `--all` child, which runs only under a prime owned here, does not carry it.
  Tests: the machinecli rows assert the exact child arguments, an orchcli row asserts a prepared bring-up under the option reconciles and claims no prime, and a beamcli test pins the hidden flag and its pass-through into the options.
- `machinecli: fix F2 F3 F4 F6 F9`:
  - F2: see the disposition above. The verb's tests run `output.Summary` over the envelope and assert the attach command, the hold's machine and subjects, and the re-run.
  - F3: an entry whose dobby outcome is refused or failed is a failed worktree and exits 1; its warnings join the hub's, prefixed with the worktree.
  - F4: `parseBeamStartOutput` takes the child's stdout, stderr and run error; a table test covers a success envelope, a partial-failure envelope, a refusal with no result, no JSON line, empty output with stderr, and empty output with a run error.
  - F6: the start tests record each terminal's directory and title and the workspace path and assert their exact values, with rows for an entry error, a failed dobby, a terminal that does not open, a workspace that does not open and a failed hub followed by a good one.
  - F9: an unknown shortname is refused with the verb's own message and way forward, the cause after it.
- `dobbyengine: fix F5 F8`: every row's board has this machine own something, so the catch-up keeps a launcher the start wrote; a row covers a beam owned here under a prime owned elsewhere; the keep marker sits above the catch-up test.
- `battencli: fix F7`: `TestHubCatchUp` asserts `Launcher` holds the hub's board directory and recorded shortname.
- `desktoplauncher: fix F10 F11 F12`: the WSL launcher refuses a double quote, dollar sign, backquote or backslash in the lym path or shortname, with rows for each; Add and Remove hold the desktop's lock; `freedesktopDesktopDir` takes the config and home directory functions, with a table test for a named desktop, the `$HOME/Desktop` fallback and an unreadable `user-dirs.dirs`.
- `pattern: fix F13` and `cmd/lym: fix F14`: the isolation background and the machine module's exclusion reason name the new variable and verb.
- A later commit adds the beamcli flag test and the field comment the self-check asked for.
- A gate repair replaced the `standalonestate.Derive` call in `desktoplauncher` with `standalonestate.UserStateDir`, since the cliwire caller-set test pins `Derive` to `internal/cliwire`.
  The F11 lock file is now `<per-user state dir>/launchers.lock`, one per user rather than per desktop directory.
- A later verify run failed `slackengine` `TestRunRelay/WatchThatFailsToOpenFallsBackToTheOnDiskCheckWithItsOwnBackoff` on a pending-wait timing check.
  This round changes nothing in `internal/slackengine`, and the package imports none of the changed packages.
  It passed on two re-runs, so no code changed for it; it reads as a flaky test.
- A gate repair rebroke two comments that wrapped mid-sentence, in `internal/desktoplauncher/doc.go` and `parseBeamStartOutput`, so each sentence sits on its own line.

Verification: `lym gate test` tier0 over `machinecli`, `desktoplauncher`, `dobbyengine`, `orchcli`, `beamcli`, `battencli` and `cmd/lym`, tier1 over `battencli` and `beamcli`, and a `GOOS=windows` build of `desktoplauncher` and `machinecli`, all pass.

## Disputed

None.

## Deferred

None.

## Self-check

Four forks returned reports, saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness, MEDIUM: no test links the `--no-prime-claim` flag to `BeamUpOptions.NoPrimeClaim`; fixed with `startOptions` and a beamcli test of the hidden flag and the options it builds.
- fix-consistency, LOW: the hub keys its shortname `name` and each worktree keys it `shortname`; fixed with a field comment naming the twin, since the hub's key is the summary's label.
- fix-proposal: no findings; a below-LOW note that the F9 refusal text covers any lookup failure, which the appended cause names, needs no change.
- fix-affected-set: no findings; it noted that `fabriccli/anchordeps.go` also runs `lym beam start <label>` as a child, which is outside this review and was not changed.

## Rulings

None.
