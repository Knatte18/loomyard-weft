# Refuse raw tmux and script in autonomous runs; learn tools from tests

A darn writer that wanted to learn how tmux resolves a client started its own tmux server and ran `(script -qfc "tmux … attach …" /dev/null </dev/zero … &)` twice.
Each `script` read zero bytes without end and queued them in memory for a client that could not take them.
The subshell detached both to the user manager, outside the memory scope lym puts each test binary in, and the machine froze.
This change closes the shell route and names the gated route in its place.

## What changes

- **Shell guard row `tmux-script`** (`internal/shellguard`).
  It refuses `tmux` and `script` as a whole word in command position: after a separator, a subshell paren, `exec` or `sudo`, path-prefixed, and inside a `sh -c` script.
  Its steer says why: such a process outlives the command and every memory cap.
  As the way forward, it names a tmux-tier test written in the repository and run through `lym gate test` with its package and a filter.
  The row carries no `Forms`: like the computed-path `rm` deny it is pattern-shaped, and `pattern/PATTERN-refusal-way-forward.md` now lists it as such.
- **New switch `SwitchAutonomous`.**
  A row hanging on it is installed and announced only for a launch whose spec is not interactive.
  `claudeengine` and `codexengine` both map it, from `denyInputs.interactive` and from a new `guardInputs.autonomous`, so the engines stay at parity.
- **Writer and fixer stencils.**
  Both darn make-change fragments tell the writer to learn how an external tool behaves from a test it writes, in the tier the hub runs that tool's tests in.
  They also forbid learning it from processes started in the shell, and forbid any background or detached process.
  In the gated fragment, such a test is a second exception to "run no `lym gate test` yourself".
  It goes through the same `{{.measure_call}}` as a golden value: one package or project and one test class, so it never starts a whole tier.
  The cold fixer frame gains a short `## External tools and processes` section with the same two rules.

## Design decisions

- **Autonomous runs only, not every run.**
  The entry asks for the refusal in every run agent's shell.
  The sampler stencil tells its operator-attended agent to start `tmux -L <name>` for a trial, so a `SwitchNone` row would contradict a shipped stencil.
  Gating on the spec's `Interactive` flag covers every writer, fixer, reviewer and driver, all autonomous, and leaves interactive sessions alone.
- **How narrow the tmux-tier opening is.**
  The record carries no Discussion, so this change settles the point the entry left to one.
  The opening is the gated writer's existing measuring call.
  With a hub whose darn-write key lists the tmux tier, that call runs one package with a filter selecting one test class.
  No gate-side check was added: `lym gate test` has no way to tell an agent's single-test call from a gate row's call, and the stencil names the bounded form.
- **No `Forms` on the row.**
  Single-token forms `tmux` and `script` would make the no-denied-recovery token scan flag prose such as "a tmux pane" and "when a script fails" across stencils and specs.

## Tests

- `TestRows_PatternsDenyAndAllow` (tier 1) gains `tmux-script` rows.
  The incident's own command and `tmux attach`, `tmux new` after a separator, path-prefixed `tmux`, `exec script` and `tmux` inside `bash -c` are denied.
  `tmux` as a grep argument, `bash script.sh`, `./script.sh`, `tmuxinator`, a quoted commit message and `lym reed attach` pass, as does the row's own description-field case.
  On the merge base the test fails at once, since the table has no `tmux-script` row.
- The Claude and Codex manifest tests expect the row on an autonomous spec and its absence on an interactive one.
- `TestBuildSettings_NoAgentHookAndForkGuard` counts one tmux-script entry for autonomous inputs and none for interactive ones.
  `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`'s Bash deny counts rise by one.
  The Codex `installedGuards` table drops the row when the switch is off.

## Merge danger

**Door:** two-way
**Blast radius:** shellguard
