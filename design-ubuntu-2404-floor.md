# Ubuntu 24.04 as a stated and checked floor: a PATTERN entry and a mechanical check

`ubuntu-2404-floor` · Note · enhancement, reed, gate

## Seen (orch, 2026-10-23)

`e634efb9` separated the reed pane tail with `"\x1f"`, written and tested on Neve (Ubuntu 26.04, a newer tmux).
On OSL-1046 (Ubuntu 24.04, tmux 3.4), tmux prints the separator as the literal text `\037`, so the reed naming tests and a dead-strand test failed in every run cut before `a1200c20` switched to a tab.
Earlier tasks already made LoomYard run on Ubuntu 24.04.
Nothing states Ubuntu 24.04 as a supported floor: PATTERN names no platform floor, and the repository has no CI.

## Operator view, not a decision (2026-10-23)

Everything must work on Ubuntu 24.04; checking that by hand all the time is a large job.

## Idea (orch, not decided)

- A PATTERN entry naming the floor: Ubuntu 24.04 with its tmux 3.4, its git and its Go toolchain, so a writer and a reviewer know what an external tool's behaviour may be assumed from.
- A mechanical check, so no one checks by hand, either:
  - a CI job on GitHub's `ubuntu-24.04` runner, tmux from apt, that runs the tiers up to the tmux tier on every push to `main`; or
  - the gate's tmux tier runs against a tmux 3.4 binary on every machine, so a run on Neve meets the floor's tmux too.
- OSL-1046 runs the tmux tier in every end gate since 2026-10-23 (then keyed `final` and `landing`, now `webster`, `merge-in` and `finalize`), so a run there already meets the floor; the gap is runs on Neve.


## From issue #848

Imported from [issue #848](https://github.com/Knatte18/loomyard/issues/848).

## What happened

Twice in one task run, the Publish row halted `blocked` (anomaly `escalation-to-human`) with "publish gate failed after merging parent branch main (exit code 1)".
Both gate runs (`lyx gate test` tier2) reported the same 8 failures, in two packages: `internal/reedcli` and `internal/reedengine`.

- `TestNaming/TaskWorktreeStrandNameTitleAndEnv`, `TestNaming/EmptySlugGivesShortnameAndRole`, `TestNaming/RepairNamesRestoresAHandChangedTitle` and their parent `TestNaming` (`internal/reedcli/naming_integration_test.go`): the pane title read back is the strand name followed by literal `\037` escapes and the shell name and path (shape `tc:helper\037bash\037/usr/bin/bash`), where the test wants `tc:helper`.
- `TestStrandNamingAndList/HubWideDirectoryAndDrift`: the same family, failing in the same package.
- `TestOwnWindow_RealTmux/ShellOnlyStrandReadsDeadPastTheGraceAndResumeRelaunchesIt` (`internal/reedengine/ownwindow_integration_test.go`): `Status` of a shell-only strand past the grace returns `Live:true`, where the test wants present and not live.

The failures were identical on the two runs, which differ only in what the Darn-Write pass changed in between.
The task itself changes the board engine, not reed.
The failures sit in code the task did not touch, so they are most likely environmental (the host tmux version's format output) or came in with the merged parent branch.

## Why it is abnormal

The recovery path the halt prescribes (`lyx loom goto --to Darn-Write`, then `lyx loom resume`) cannot fix a failure outside the task's files.
The loop was walked once and re-blocked at Publish with the same 8 failures, so the run needs the operator for a failure that no task change can clear.
The real-tmux tests also look host-sensitive: pane titles that include the `\037` separator suggest reed's title format string is not interpreted as a control character by this tmux, and a shell-only pane is judged alive past the grace.

## What to do

- Determine whether these real-tmux reed tests fail on the parent branch itself on this host, and if so fix them at the root: stop depending on a tmux format escape that this tmux version emits literally, and re-check the shell-only liveness rule.
- Consider having the Publish gate tell a failure the task introduced from one the merged parent already carries, so the halt note names it and does not send the run back to Darn-Write.

Provenance: task slug `prime-owner-visible`; notes `loom-halt.md`, `loom-halt-2.md`.
