# Read deny_python live in the python guard hook

Turning `deny_python` off for a hub now reaches every running session at its next matching call, so no session or watcher needs a restart.
Before, each engine fixed the value into a session's hook at launch, and a session launched before the change kept refusing Python.

## What changes

- **Every session installs the python guard's hook.**
  Claude and Codex both install the python row in every launch, whatever the launch's `deny_python`.
  On a command that matches the row, the hook runs the new hidden verb `lym shuttle hook-python-allowed` and denies the call unless the verb exits 0.
  A command that does not run Python never starts lym, so the guard adds no work to other calls.
- **`lym shuttle hook-python-allowed`** (`internal/shuttlecli/hookpython.go`) resolves the worktree from its working directory with `lymcwd.ResolveWithin`.
  It loads the shuttle config through `fabricengine.ConfigDirs`, the same layers `lym config --resolved shuttle` reads, and exits 0 only while `deny_python` is false.
  It skips the shuttle group's shared pre-run, prints nothing on standard output and reports a failure on standard error.
  A failure exits 1 like a deny, so a hook that cannot ask the hub keeps denying.
- **Where the hook runs the verb.**
  The Claude hook runs it from `$CLAUDE_PROJECT_DIR`, as the creel mail hook runs lym.
  The Codex hook runs it from the hook's working directory, the session's own.
  Both send the verb's output to `/dev/null`, so the hook prints the deny or nothing.
- **The system-prompt notice still follows the value at launch.**
  Each engine now keeps two switch maps, one deciding whether a row is installed and one whether it is announced.
  The python switch is installed always and announced only when `DenyPython` was true at launch.
  Claude's `standingDeny` gains an `announced` predicate, and Codex's `denyNotice` takes the guard inputs to filter the rows it announces.
- The package docs of `claudeengine`, `codexengine` and `shellguard`, the `DenyPython` field doc, the template's `deny_python` comment and `docs/overview.md` now describe the live read.

## Design decisions

- **The launch path was not the bug.**
  The board entry suspected that `lym orch` loads the shuttle config from different dirs than `lym config` resolves.
  The orch wiring calls `shuttleengine.LoadConfig(fabricengine.ConfigDirs(location), "shuttle")`, the same dirs `lym config` uses.
  In the odd hub, the orch launch at 08:35 on 2026-11-18, after `deny_python: false` was committed at 08:33, carries no python sentence in its `append-system-prompt.txt` and no python row in its `settings.json`.
  The refusals the operator saw came from sessions launched before the change, whose hook still held the old value.
  The parent's added decision covers that case, so no change to the launch's config loading was made.
- **The board entry's test is replaced.**
  The entry's Idea asked for an orch launch with `deny_python: false` to carry neither the notice nor the hook row.
  Under the added decision the hook row stays and reads the value live.
  The tests pin the notice's absence and the hook's live answer instead.
- **The guard fails closed.**
  A missing lym, a cwd the verb cannot resolve or a config that does not load all deny, as the guard did before the change.
- **The guard command stays a pure function of the row.**
  The verb finds the hub from the hook's working directory, so the rendered command holds no path.
  Each engine renders it from the row alone, and its manifest proofs need no new inputs.

## Tests

- `TestHookPythonAllowedVerb_FollowsTheHubsDenyPython` runs the verb over a `hubforge` hub.
  It sets `deny_python` at the default layer to false, then true, then false again, and expects exit 0, 1 and 0.
  It also expects exit 1 from a directory outside any worktree.
- `TestShellGuardDenies_PythonRowAsksTheHubOnAMatch` (Claude) and `TestGuardCommand_PythonRowAsksTheHubOnAMatch` (Codex) run the rendered hook under `sh` with a stub `lym` on PATH.
  A python command asks lym from the expected directory and is denied unless lym exits 0.
  It is denied with no lym on PATH, and a command that runs no Python never starts lym.
- The guard matrices, the manifest tests and the settings golden now expect the python row in every launch.
  The deny-notice tests expect its sentence only while `DenyPython` is true.
