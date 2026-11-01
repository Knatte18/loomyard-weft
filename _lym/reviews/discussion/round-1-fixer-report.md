# Fixer report — discussion, round 1

Every finding's premise holds against the code read: `lymcwd.Resolve`'s anchor gate, `stripAgentNameEnv` in `internal/reedengine/lifecycle.go`, `reedengine.LoadConfig`'s typed unmarshal, `ValidateMachine`'s mapping-or-not check, `ParentOf` and `NamedWorktree` building absent paths, `ResolveBattenPrime`'s landing refusal, and the server spawn's `-c e.geom.PaneCwd`.

## Changes

- F1 (fixed as proposed): D4 defines `{worktree}` as the session worktree's anchor path (`AnchorPath`, which `PaneCwd` equals) and `{beam}` as the beam's anchor path, read for a task through `lymcwd.NamedWorktree`'s `AnchorPath`. The `Resolve` evidence is added to D4's rationale. D5's spec and D7's run directory now hold anchors (D7 as reshaped by F6). The D8 criterion names the beam's anchor. The auto-mode assumption, the support log's Q2 and the rejected `{beam}`-slug alternative now say anchor.
- F2 (fixed as proposed): D7 drops the env removal. It now states that the server spawn already strips both variables (`stripAgentNameEnv`), so a click reads as the operator and a direct call keeps its caller's identity. The Bound names the direct `lym reed button --confirmed` call, and the sentence "only an operator's click runs it" is deleted. The support log's Turn 1 fact is corrected, and Q8 is updated.
- F3 (fixed as proposed): D7 adds that `lym reed button` exits 0 on every path, on both lines, and cites the tmux 3.6 view-mode probe. The D7 criterion now says no click, refused or failed, opens a view-mode pane.
- F4 (fixed as proposed, then narrowed by the self-check): D2 says both `Config` types decode `buttons` as an untyped node, and the render skips a wrong-type group or entry with the same Warn. It names `reedengine.LoadConfig` and `desktophook.LoadConfig` with `afterWindow`. The D2 criterion is extended.
- F5 (fixed as proposed): D4 leaves out `{beam}` entries, with one Warn, when the beam's anchor directory is absent on this machine, and the rationale gives the `ParentOf` and `NamedWorktree` evidence. D8's rationale covers the landing refusal reaching the operator as D7's failure message. The D8 criterion covers a present, non-landing beam only. The D4 criterion covers the absent beam.
- F6 (fixed differently): D7's `run-shell -c` now reads a new session option `@lym_button_dir`, rendered with the others (D5). The run directory is the beam's anchor in a task session, the prime's anchor when that beam is absent, and the session's own anchor elsewhere. D7 notes that the verb's log lands there (`internal/logger/logger.go:48`: the trace file sits under the current worktree's anchor). CANCEL drops `cd {beam} &&`, and `{worktree}` still names the task for `term`.
  - Not made: running every task command from the beam with no further change. `lym shuttle open` resolves its location from its cwd (`internal/shuttlecli/cli.go:76`), so the `agent` button on a task session would have opened the agent in the beam.
  - Made instead: `agent` ships as `cd {worktree} && lym shuttle open …`, a no-op outside a task session.
  - The prime fallback was added because the proposal left the run directory of a task whose beam is absent unset. `ResolveBattenPrime` accepts the prime.
- F7 (fixed as proposed): D2 bars `%` in `label` and `confirm`, and its rationale cites the probe. D6 doubles each `%` as well as each `#`. The D6 criterion is extended.
- F8 (fixed as proposed): D4 says a failed desktop load inside the closure returns no machine buttons and leaves the place and values intact. Only a failure to read the place or a value shows `follow` alone. The D4 criterion follows.
- F9 (fixed as proposed): the closure returns the place, the three cheap values, the machine's buttons and a separate `{beam}` function. The render calls that function when the composed row uses `{beam}` or the session is a task, since F6 makes `{beam}` a task's run directory.
- F10 (fixed as proposed): D7's Bound names `reed.yaml`'s hub base and worktree layers and `desktop.yaml`'s hub base and machine layer.
- F11 (fixed as proposed): D1's Bound says any session an older binary boots or attaches keeps `status 2` until the newest binary next attaches it.
- F12 (fixed as proposed): the `agent` entry quotes `'opus[high]'`, and D8's rationale says why.
- The support log's Review rounds section records round 1.

## Self-check

- fix-consistency, LOW: Q7 still listed `#` and control characters only. Fixed as proposed: Q7 now lists `%`.
- fix-consistency, LOW: Turn 3 still recorded the old `#{session_path}` and output-discard resolutions. Fixed as proposed: both entries now note what round 1 changed.
- fix-consistency, LOW: D7 said "a client or worktree gone" after CANCEL. Fixed as proposed: it now says "a client gone".
- fix-claims, MEDIUM, merged with fix-proposal MEDIUM 1: D2's "keeps whole every load" overclaimed, since the layers' shape check refuses a non-mapping `buttons` key or a group written as a mapping before any decode. Fixed as proposed: D2 narrows the guarantee to wrong types the shape check admits and states the refused shapes. The D2 criterion follows.
- fix-proposal, MEDIUM: `{worktree}` as an anchor makes `term`'s default window title the anchor's base name. Fixed as proposed: `term` ships with `--title {slug}`, and D8's rationale says why.
- fix-way-forward, LOW: the exit code was unstated when `display-message` itself fails. Fixed as proposed: D7's exit-0 list now includes a failed `display-message`.
- fix-way-forward, LOW: a beam removed after the render leaves the run directory stale. Fixed with my own change: the spec also carries the run directory (D5). When that directory is gone, the verb shows a status message naming it and saying the row re-renders at the next attach, and runs nothing. A gone run directory joins the exit-0 list.
- fix-claims, out of scope, `run-shell -c` format expansion is undocumented: no change. The record relied on it before this round with `'#{session_path}'`, and the fix did not introduce the assumption.

## Disputed

None.

## Deferred

None.
