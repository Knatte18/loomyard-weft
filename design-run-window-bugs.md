# Run window and shuttle bugs from 2026-10-12

`run-window-bugs` · Task · bug, enhancement, shuttle, reed, loom, fabric · running · Discussion-Bouncer

This task bundles the run window and shuttle bugs and fixes noted on 2026-10-12: `shuttle-ready-blank-pane`, `loom-checklist-stale-render`, `weft-push-refused-retry`, `shuttle-color-verified-send`, `reed-socket-test-fd-sweep` and `agent-gofmt-verb`.

## Was `shuttle-ready-blank-pane`

Shuttle's startup probe decides readiness only from a pane capture (`internal/shuttleengine/wait.go`, through `claudeengine/startup.go`), looking for the prompt glyph or `shortcuts` on screen.

On 2026-10-12 convergence-plan's parent-review alignment agent (run dir `4e8c64f5…`) started, took its prompt and was mid-review when shuttle marked it died at `startup_timeout_s` (90 s) and removed its strand.
`startup-capture.txt` held a single newline: the pane was blank.
The likely cause is reed's layout: loom status, driver, Discussion-Write and the alignment strand exceed the 44-row window, so `clampToFit` (`reedengine/render/height.go`) floors a pane to about one row.
The review fell back to the orch, so the verdict path held but ran without the alignment pre-pass.
It was the first use of the alignment agent after `alignment-agent` landed.

## What to do

- Treat a recorded `SessionStart` plus a `UserPromptSubmit` or tool event in the run's `events.jsonl` as proof of readiness, ahead of the capture.
- Give the probe a minimum pane height, or capture scrollback, so a collapsed strand cannot read as never-ready.

## Seen again

- 2026-10-12, `operator-followups` parent review round 1: the alignment agent died at spawn the same way (blank capture, "provider never became ready").
- `operator-followups`' D6 halves the loom status pane's share of the window, which eases the squeeze but does not fix the probe.

`reed-multi-window` lands first and gives agents their own windows; re-check the readiness probe against that layout, but keep the fix: readiness must not depend on a pane capture alone.

## Was `loom-checklist-stale-render`

The operator saw two loom checklist renders stacked in one run window's status pane on 2026-10-12 (task `run-robustness`): an older `loom paused | now Webster-Burler-Review` block, with a right-hand column left over from a wider layout, above the current `loom running | now Merge-In` block.

The old render is not cleared when the new one is drawn, so the pane mixes two states and reads as if boxes were assigned at random.

## What to do

- Clear or overwrite the whole pane on every redraw, including after a resize or a layout change from two columns to one.
- Pin it with a test that redraws a shorter, narrower checklist over a longer, wider one.

## Was `weft-push-refused-retry`

On 2026-10-12 the weft records push at Publish was refused once and passed on retry in three runs: `board-field-drop`, `webster-pack-read-set` and `alignment-agent`.

A refusal that clears on retry points at a race on the weft remote, likely another run's push or a board write landing between fetch and push.
The retry hides it today, but each refusal costs a round and a run without retry headroom would halt.

## What to find

- Which ref was refused and why (non-fast-forward or lock), from the Publish trace in each run.
- Whether the refusal comes from concurrent runs writing weft at the same time.
- Whether the push should fetch and rebase before its first attempt, or the retry is the intended path and only needs logging.

## Seen again

2026-10-12 16:04, `lyx batten run intake-bugs-1012` while the orch was writing many board entries in a loop: `gitrepo: push refused, retrying ... ref=main attempt=1 stderr="! [remote rejected] main -> main (failed)"`.
The stderr is `remote rejected ... (failed)`, not a non-fast-forward rejection, which points at the remote refusing concurrent ref updates rather than a stale local ref.

## Was `shuttle-color-verified-send`

## Why

Shuttle types `/color <color>` into a fresh session's pane, and with skills it types the skill-load message and then the prompt as well.
Every typed send is a chance for a lost or half-landed message, and the operator finds the typing clumsy (2026-10-08).

## Facts (Claude Code 2.1.294 CLI reference)

- No CLI flag, settings key or environment variable sets the session color; `/color` in the session is the only way.
- The positional `claude "<text>"` submits its text as the first user turn of an interactive session, and a slash command or skill invocation works there.
- Only one positional prompt exists, so the launch line carries one first turn.

## Proposal

- The launch line carries the first turn in every case: the skill-load message when skills are named, else the prompt, so only the turns after the first are typed.
- `/color` stays typed, as there is no launch-time alternative, but goes through shuttle's verified send once `shuttle-send` lands, after the session is ready.

## Priority

Operator, 2026-10-08: low; the typed skill load is robust, and the launch line saves only a little time.

## Done on 2026-10-08

`/color` goes through shuttle's verified send after the session is ready (main e12fb6afb, with the settle wait from 7b3d140cb).
What remains is the first proposal bullet: the launch line carries the first turn.

## Was `reed-socket-test-fd-sweep`

Seen in shuttle-startup-ready (PR #557, 2026-10-11): `TestRemoveStaleSocket` in `internal/reedengine` failed at Publish verify because a parallel test's forked child inherited the closed listener's fd, so the "stale" socket still answered.
Commit `ea4e53819` fixed `TestRemoveStaleSocket` and `TestRemoveSocketFileOnceGone` by building the stale socket from a datagram socket that never listens (`bindStaleUnixSocket`).

Idea: sweep other tests that build a stale socket by closing a listener and move them to `bindStaleUnixSocket`.

## Was `agent-gofmt-verb`

Imported from [issue #578](https://github.com/Knatte18/loomyard/issues/578).

An implementing agent ran `gofmt -w` once on a test file to fix a map literal's formatting, and recorded that this was against the agents' no-rewrite-script rule.
It only reformatted the literal, but the agent had no sanctioned way to format Go files: Edit and Write are the only permitted file changes, and no `lyx` verb formats.

This is abnormal because the rule and the tooling conflict: formatting-only fixes force either a rule breach or manual whitespace edits.
What to do: provide a `lyx` formatting verb (or gate step) that agents may run, or state in the agent prompt that `gofmt -w` on a single file is allowed.

Provenance: task `merge-review`; notes 18-merge-in-row.md.

## Teardown races a reed lock

2026-10-12 17:01, `operator-followups` Worktree-Teardown: `git worktree remove` failed with "Directory not empty".
After removal had begun, something recreated `.lyx/reed.lock` and `.lyx/reed.json.lock` (both empty, same second) in the pair, so the removal could not finish.
The likely writer is a reed process still bound to the pair: the run window's `lyx reed attach`, or a status or resize hook firing as the session closed.
The orch removed the two empty lock files and re-ran `lyx batten run operator-followups`, which finished the teardown.
What to fix: teardown waits for, or kills, every reed process bound to the pair before it removes the tree, or reed refuses to create its lock files in a pair that is being torn down.
