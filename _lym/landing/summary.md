# Refuse raw tmux and script in run sessions; narrow the tmux-tier gate

A darn writer that wanted to learn how tmux resolves a client started its own tmux server and two `script -qfc "tmux ... attach" /dev/null </dev/zero` processes from Bash.
Each `script` queued the endless zero stream in memory, and both outlived the Bash call outside the memory scope lym gives each test binary, so the machine froze.
The safe path was closed and the dangerous one open: the agent was refused tmux-tier `lym gate test` runs, yet raw `tmux` and `script` in its Bash passed.

## What changes

- `internal/shellguard` gains a `tmux-script` row that refuses `tmux` and `script` as a command word, optionally path-prefixed.
  The refusal names the way forward: write a tmux-tier test in the repository and run that one test through `lym gate test --tier`, with its package and a filter after `--`.
- The row sits behind a new `deny_raw_tmux` switch.
  Both the claude engine (`settings.go`) and the codex engine (`guards.go`) install and announce it for every run session, interactive ones included, unless the launch spec sets `AllowRawTmux`.
- `samplerengine` sets `AllowRawTmux`, because its stencil has it start `tmux -L` servers for its trials.
  The exemption is a named field on the spawn spec (`shuttleengine/spec.go`), not a check of `Interactive`.
- `lym gate test` refuses a run agent's call that lists the tmux tier, through `--tier` or an `--upto` range reaching it, and names no package target.
  A wildcard such as `./...` names none.
  The refusal names the single-test form as the way forward.
  A run agent is recognised by the strand name in its environment; a call from outside a run is unaffected.
- The writer and fixer stencils (darn, webster, weaver, burler, landing, gatefix, loom arbiter fix) tell the agent to learn how an external tool behaves from a test in the repository under the gate, and never to start a background or detached process from Bash.
  The darn writer's one measuring run now also covers such a fact about an external tool.
- Docs follow: `shellguard`, `gatecli` and `claudeengine` package docs, and `pattern/PATTERN-refusal-way-forward.md`.

## Design decisions

- Item 1 of the board entry says every run agent's Bash, so the row reaches interactive sessions too, not only unattended writers.
- The opening in item 2 is enforced in the gate, not in prose: a stencil sentence alone would not stop a run from starting the whole tmux tier.
- The shell row is a guardrail, not a barrier, as the row's comment states.
  A launch behind a script file, `eval`, `nohup`, `setsid` or `timeout` passes, and so does one spelled through a variable.
  A program named `script` or `tmux` under another directory, such as `tools/script`, is falsely denied.
- The claude settings golden (`testdata/settings-golden.json`) and the engine manifest and prepare tests change because the new row is installed in every session.

## To check by hand

- Start a run session and run `tmux ls` and `script -qfc true /dev/null` in its Bash: both should be refused with the way forward.
- In a sampler session, run `tmux -L x ls`: it should pass.
- From a run agent, run `lym gate test --tier <tmux tier>` with no package and then with a package and a filter after `--`: the first is refused, the second runs.

## Merge danger

**Door:** two-way
**Blast radius:** shellguard
