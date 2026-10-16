# Support log — reed-down-and-pane-path

## Interview

The session ran autonomously with no advisors; every turn below is a self-posed question and its auto-pick.

### Turn 1 — what does #679 ask to end?

Q: the watchdog is per hub, not per worktree; should `down` end it whenever it runs, or only when the hub is left with no session?
Recommended: only when `down` ended the hub's server, because siblings still need name repair.
Picked: recommended (D1, D3).

### Turn 2 — why did the watchdog outlive `down` and then respawn?

Exploration: the daemon exits only after three consecutive idle cycles on a backed-off cadence, so it lingers well after `down`.
Its binary-replaced check runs before the listing and always spawns a successor, so a deploy during that linger revived it on the new binary.
Q: fix the linger, the respawn, or both?
Recommended: both — a stop request from `down` (D1, D2) and a respawn only for a live hub (D4).
Picked: recommended.

### Turn 3 — how does `down` reach the daemon?

Options: a fixed marker in the existing discover signal file; a new stop file with its own watch; a pid in the lock file and a signal.
Recommended: the marker in the discover signal file, already watched, cross-platform, and naturally undone by a racing cold boot's truncate.
Picked: recommended (D1).

### Turn 4 — should `down` wait for the exit?

Recommended: yes, bounded at 10 seconds on the lock, warn and succeed on timeout, so the "no process remains" expectation holds without making `down` fallible.
Picked: recommended (D3).

### Turn 5 — where does #680's PATH get reordered?

Exploration: reed's prelude is sourced in the pane shell after that shell's rc, so the pane shell resolves correctly.
The operator's `~/.bashrc` prepends `~/go/bin`; a fresh shell below the pane shell re-runs it: a `--cmd bash` strand, and Claude Code's Bash tool through its shell snapshot.
`LYX_BIN` survives, since it is inherited and no rc touches it.
Q: fix at reed level for every child shell, or in the Claude engine for lyx's agents?
Recommended: the Claude engine's existing `CLAUDE_ENV_FILE`, which runs after the snapshot before every Bash command; reed has no engine-neutral hook that runs after a child shell's rc.
Picked: recommended (D5, D7).

### Turn 6 — where does the `LYX_BIN` key live?

Recommended: exported from `internal/agentname` beside the other strand env keys, which the Claude engine already imports.
Picked: recommended (D6).

### Turn 7 — what do the suites check?

Recommended: each suite's pre-condition names the shell where the guarantee holds; the creel suite's bash strands use `bash --norc`, and its plain Claude strands are named as outside the guarantee.
Picked: recommended (D8), later narrowed by the lens review to the pre-condition bullet only.

### Turn 8 — lens review

Six forks reviewed the record; every one returned a report.

- premises, MEDIUM, pair teardown's `EndSessionByName` skips `Down`: fixed, named as a Scope exclusion that falls back to the idle rule.
- premises + scenarios, MEDIUM, a session booting while the daemon exits loses the lock race: fixed, D3a re-check after lock release.
- scenarios, MEDIUM, a transient list-sessions error counts as idle under D2 and D4: fixed, bound stated and closed by D3a; restricting to exit-0-empty was rejected because the normal post-`down` listing is the no-server error.
- scenarios, MEDIUM, `down`'s lock probe can block a spawning successor: fixed, bound stated in D3.
- scenarios, MEDIUM, a failed file watch never sees the stop: fixed, stated in D2's bound as falling back to the idle rule.
- pattern-fit + board-fit, MEDIUM, the stop bypasses `PATTERN-daemon-wakeups`' floor: fixed, stop wakes keep a 1-second floor.
- goal-scope, MEDIUM, #680's narrowing reads as an interpretation: fixed, Scope states it as a departure with its reason.
- goal-scope + board-fit + leanness, MEDIUM/LOW, the reminder wording handed to `creel-stage1-fixes`, which excludes it: fixed, now a plain exclusion with no owner claimed.
- goal-scope + board-fit, MEDIUM, `bash --norc` in the creel suite dodges the defect and overlaps `creel-stage1-fixes`: fixed, D8 limited to the pre-condition bullet; overlap named in Open risks.
- goal-scope, LOW, D5 rests on unverified Claude Code ordering: fixed, live shuttle-suite criterion added.
- premises, LOW, `consumeDiscoverSignal` removes without reading: fixed, plan-writer note and D2 wording.
- premises + pattern-fit, LOW, Windows content and the shell-seam hedge: fixed, per-OS constant with `cygpath -u`, seam stated as not binding.
- scenarios, LOW, `Down`'s failure path: fixed, D1 writes the stop either way and D3 waits only on success.
- scenarios, LOW, stale marker, quoting, PATH growth and in-flight reap: fixed, criteria and an assumption added.
- pattern-fit, LOW, the 10-second bound needs a seam: fixed, D3 makes it injectable.
- pattern-fit, LOW, `agentname` stretch: fixed, D6 accepts it with its reason.
- board-fit, LOW, a future engine would not get the PATH fix: fixed, D7's doc line names the per-engine rule.
- board-fit, LOW, `rename-binary-lyd` would rename the key: rejected, no conflict now and D6 keeps it to one site.
- leanness, MEDIUM/LOW, repeated child-shell boundary, generic constraints, duplicate 10-second assumption, test-budget note, doc-check criterion: fixed, cut.
- leanness, LOW, the half-argued rejected alternative: fixed, restated.

## Rejected alternatives

- End the watchdog on every `down`, siblings or not: drops name repair for live sibling sessions.
- A separate stop file with its own file watch: a second watch for what the discover signal already carries.
- A pid in the lock file and a kill signal from `down`: adds a pid record that can go stale and a signal path that differs on Windows.
- Count the stop toward the idle cycles instead of exiting at once: still leaves `down` waiting through backed-off cadence.
- Keep the replaced check first and suppress only the respawn: before the listing it cannot tell a live hub from an idle one, so it would either leave a live hub without a successor or keep respawning for an idle one.
- Fast exit and no-respawn only on an exit-0 empty listing: the normal post-`down` listing is a no-server error, and psmux cannot tell the two apart.
- Switching the creel suite's bash strands to `bash --norc`: dodges the defect, changes what those strands stand in for, and collides with `creel-stage1-fixes`' suite edits.
- An exported bash function `lyx` in the prelude: bash-only, misses zsh and pwsh, and `command -v lyx` would print `lyx` instead of a path.
- An exported `PROMPT_COMMAND` re-prepending PATH: interactive bash only, never reaches Claude's Bash tool, and an rc that sets its own drops it.
- Reed exporting `CLAUDE_ENV_FILE`: puts a provider specific in reed, against `PATTERN-shuttle-provider-seam`.
- Prepare writing the binary's directory as a literal into `bash-env.sh`: duplicates what the pane prelude already exports and goes stale if a resume runs from another binary.

## Review rounds

_No rounds yet._

## Question ledger

- Auto-pick: the watchdog ends only when `down` ended the hub's server (Turn 1).
- Auto-pick: fix both the linger and the idle respawn (Turn 2).
- Auto-pick: the stop marker rides the discover signal file (Turn 3).
- Auto-pick: `down` waits on the lock, 10 seconds, warn on timeout (Turn 4).
- Auto-pick: the PATH fix goes into the Claude engine's env file; reed's prelude is unchanged (Turn 5).
- Auto-pick: `LYX_BIN`'s key moves to `internal/agentname` (Turn 6).
- Auto-pick: suite pre-conditions reworded per D8 (Turn 7).
- Auto-pick: an exiting daemon re-checks for sessions after releasing its lock (Turn 8, D3a).
- Auto-pick: stop wakes keep a 1-second floor (Turn 8).
- Resolved by reading: the operator's `~/.bashrc` prepends `~/go/bin` and `~/.local/bin`, matching the issue's observed PATH.
