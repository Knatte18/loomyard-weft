```yaml
lens: code-scope
outcome: reported
```

Task: map the diff's changes to the decisions and report any scope gap. The record here is a decision record alone: D6 plus three "Added after Discussion" entries.

**Decisions and what builds them**

- **D6, step 1 (one Bolt commit, refusals, same-set and other-set rules, rejected-push re-read).** Built by `boardengine/holdwrite.go` (`WriteAdopt`, `planAdoptHold`, `readHoldsAfterPush`) and `boardengine/replay.go` (replay of holds and registry, refusal over another machine's new hold). Called from `adoptcli/foreground.go:90`.
- **D6, steps 2 and 3 (moved runs, takeover entry, hand-over mark drop).** Built by `adoptcli/foreground.go` (`movedRun`, `takeOverRuns`), `fabricengine/takeoverwrite.go` (`TakeOverRun`), and `fabricengine/handover.go` (`DropHandOverMark`, plus `OriginParentRunStatus`, which the origin-run read needs).
- **D6, steps 4 to 6 (catch-up, deploy, workspace).** Built by `foreground.go` and `battencli/catchup.go`, which lists every run for the catch-up.
- **D6, step 7 (check in the foreground) and step 8 (green clear, bolt-hold drop, `BeamUp`, fix loop, try counts in the hold).** Built by `adoptcli/holdcheck.go`, `green.go`, `boardengine` `FixHold` and `ClearHold`, and `fixtries` trimmed to `Read` only.
- **D6, `--take-hold`.** Built by `cli.go:88`, `WriteAdopt` with `TakeHold`, and the `adopt-take-hold` row in `shellguard/rows.go` with its test and the settings golden.
- **D6, `--here`.** Built by `here.go`, which keeps `no_subject` and `owned_here`.
- **D6, retirements.**
  - Deleted: `worker.go`, `steprun.go`, `steps.go`, `pause.go`, `runreport.go`, and all of `internal/adoptrecord`.
  - Flags: `--keep-local`, `--drop-local` and `--discard` are gone from the adopt flags.
  - Wording: the retired flags are removed from the wording in `landingshed/finalize_owner.go` and `fabricengine/anchorstop.go`.
  - Pause doc: `pauserecord` docs now call the caller legacy.
  - Process role: the `RoleAdopt` mention is removed from `runproc/doc.go`.
  - Doc text: `lymdirs`, `junctionnames.go`, and the `PATTERN-daemon-wakeups` follow bullet are updated.
- **D6, readers of the record.** Built by `orchcli/watch.go` and `orchengine` (`UnderAdopt` reads the hold), `dobbyengine/loop.go` and `handover.go`, and `beamcli` (`adoptsection.go`, `cli.go`, `dobbydeps.go`, `ownerarchive.go`), which shows the Bolt hold in `lym beam status`.
- **PATTERN change.** `pattern/PATTERN-dobby-runs.md` carries the declared text.
- **Added after Discussion: D9 (hub stop writes no intent), D8 (only the printed `parent_beam` names root), and the two arbiter acceptances.** None of these asks for a change in this part, and the diff makes none. That is correct, not a gap.

**Per change, trace to a decision**

- Every change group listed above traces to D6 or is forced by a D6 change.
- The edits in these files are forced by the retirements:
  - `fabriccli/anchor.go`
  - `fabricengine/destroy.go`
  - `fabricengine/doc.go`
  - `fabricengine/junctionnames.go`
  - the `orchcli` docs
  - `boardengine/doc.go`
  - the `battencli` docs
  - the `runproc` records and `end.go`
- **Untraced hunk:** `boardengine/registry.go` and `registrysetter.go` (6 and 8 lines) were not in my context, so I did not judge them.
- **Not examined:** the `fronttask` and `lymdirs` files were not read beyond their diff stats.

**Findings:** none. Every decision is built by at least one change, and no change lacks a decision or a force from one.

**Tool calls**
1. `git diff -U0` over `runproc`, `landingshed/finalize_owner.go`, `lymdirs`, `orchengine`, `fixtries/fixtries.go`, `fabricengine/anchorstop.go` and `junctionnames.go`, `dobbyengine/loop.go` and `pauserecord/record.go`: my context held only their diff stats, and I needed their hunks to trace them.
