# reed-tmux-conf — decision record

## Goal

Give the operator a way to put their own tmux configuration (prefix, key bindings, `escape-time` and the rest) back on reed's per-hub tmux servers, which since reed-hygiene (c6dc1fffc) start with `-f /dev/null` and read no `~/.tmux.conf`.
Reed's servers stay hermetic by default; the operator opts in by naming a file.

## Scope

In:

- A new `reed.yaml` key, `tmux_conf`, naming a tmux config file that reed sources into a hub server it has just started.
- Path validation for that key at boot.
- The ordering of that source against reed's own option pins, so reed's pins win.
- Test-kit hygiene so a developer's environment cannot make a test server source the developer's config.
- Docs: the template comment, `internal/reedengine/doc.go`'s "Operator config" paragraph, and `pattern/PATTERN-test-isolation.md`'s "Hermetic servers" section.

Out:

- Dedicated `reed.yaml` keys for individual tmux settings (prefix, `escape-time`, bindings).
- Re-sourcing the file into a server that is already running.
- Windows (psmux): boot, probe and spawn stay unchanged there.
- Reading `~/.tmux.conf` or the XDG path implicitly; reed sources only a file the operator names.

## Decisions

### D1 — One `tmux_conf` key in `reed.yaml`, opt-in, env-overridable

Decision: add `tmux_conf: ${env:LYX_REED_TMUX_CONF:-}` to both `reed.yaml` templates (POSIX and Windows), with a matching `Config` field.
Empty, the default, means no operator config: reed's servers behave exactly as they do today.
A materialized `reed.yaml` lacking the key takes the empty default until `lyx config reconcile` writes it.
Rationale: the operator's settings are arbitrary tmux commands, so one pointer to a file covers all of them where per-setting keys would cover a guessed few.
Opt-in keeps reed-hygiene's hermetic default, which that task chose deliberately.
The env form follows `LYX_REED_MOUSE`, `LYX_REED_SHELL` and the other reed keys, so an operator sets it once in their shell profile without committing a personal path into a repo's tracked `_lyx/config/reed.yaml`.
Bound: this adds an edge through which arbitrary operator tmux commands reach reed's server; D3 and D4 bound what it can override and when it runs.

### D2 — Path rules: `~/` expanded, absolute required, file must exist

Decision: reed expands a leading `~/` to the user's home directory, then requires an absolute path to an existing regular file.
A relative path, a missing file or a non-regular file is refused when `lyx reed up` (or any booting op) validates its config, at the same point and in the same way an unresolvable `shell` is refused today, naming the `tmux_conf` key, the `LYX_REED_TMUX_CONF` variable and the path.
The check runs whenever the key is non-empty, on every POSIX boot, whether or not that boot starts the server.
Rationale: a named file that does not exist is a configuration error, like an unresolvable `shell`, and a relative path has no meaningful base for a server whose cwd is the logs directory.
Checking on every boot keeps validation in one place instead of splitting it on whether a server happens to be running.

### D3 — Sourced once, at the server's start, before reed's pins

Decision: on POSIX, the boot that starts the hub server (the `serverSpawnArgv` path) runs `source-file <path>` against that server right after the spawn and before `pinBootOptionsLocked` and the geometry pins.
The server spawn and the capability probe keep `-f /dev/null` unchanged.
A boot that joins an already-running server sources nothing, and nothing re-sources the file into a running server.
Rationale: sourcing through a separate command keeps the probe's transient server and the spawn hermetic, so a config that creates a session cannot keep a transient server alive (the reason reed-hygiene gave the probe `-f /dev/null`).
Sourcing before the pins means every option reed sets, `-g` or per session, overwrites a conflicting value from the file.
`source-file` returns its errors to the caller, where `-f` at spawn would show them in a view-mode pane on the first session.
Bound: the file can change only what reed does not pin.
A key binding, hook, server option or session-level default reed leaves untouched stays as the file sets it; this is the point of the feature, and the operator owns its consequences.
A value change takes effect at the hub server's next start, the same rule `debug_log` and `mouse` follow.
In a hub, the worktree whose boot starts the server decides which file is sourced; another worktree's value is ignored, as its `tmux` value already is.

### D4 — Sourcing errors warn and never fail the boot

Decision: when `source-file` exits non-zero (a syntax error or a failing command inside the file), reed logs a Warn record carrying the path and tmux's stderr, prints the same warning to the booting command's stderr, and continues the boot.
Rationale: tmux applies every command in the file it can and reports the rest, so the server is usable; a typo in a personal config must not take a hub down.
The geometry pins already follow the same non-fatal rule.
Bound: only the source step is non-fatal; the path validation in D2 still refuses.

### D5 — Reed pins the server-lifetime options an operator config could break

Decision: on the server-start path, after sourcing, reed pins `exit-unattached off` on the server and `destroy-unattached off` on the session, unconditionally, so that the boot never depends on tmux defaults an operator file can change.
Rationale: either option set `on` in the file would end reed's detached hub server or session, and reed's watchers, panes and daemon depend on them staying up.
Pinning them always, not only when a file is sourced, keeps one boot path.
The plan writer adds any other option reed's correctness depends on to the same pin set, if it finds one an operator file could flip.

### D6 — Test servers never source a developer's config

Decision: `tmuxkit.Main` unsets `LYX_REED_TMUX_CONF` in the test process's environment, beside the `TMUX_TMPDIR` and `TMPDIR` it already sets, so a `reed.yaml` rendered from the template resolves `tmux_conf` to empty in every tagged test package.
A test that exercises the feature sets the key explicitly to a file it wrote.
Rationale: `PATTERN-test-isolation` requires tests to reach no operator global state, and the `${env:...}` default makes the developer's shell environment an input otherwise.

### D7 — Windows ignores the key

Decision: on Windows the key is parsed but neither validated nor sourced, and the Windows template comment says so.
Rationale: reed-hygiene left Windows boot unchanged, psmux reads its own config at start, and its `source-file` support is unverified.

## Constraints

- `PATTERN-test-isolation`: its "Hermetic servers" section states that a server reed starts reads no `~/.tmux.conf`.
  That stays true by default; the section gains one bullet saying a server reed starts sources the file `tmux_conf` names, which is empty in tests per D6.
  This changes the background file's text, not the invariant line in `PATTERN.md`.
- `PATTERN-config-strictness`: the new key goes through the existing `reed.yaml` load path; no new loader.
- `PATTERN-hub-wide-config`: `reed.yaml` stays a per-worktree module.
  `tmux_conf` is server-scoped like `tmux` and `debug_log`, which follow the first-booter rule already; making it hub-wide would move the whole module, which this task does not need.
- `PATTERN-shell-mechanics-seam`: the `source-file` call is a tmux argv built in `reedengine`, not a pane-shell string, so it does not touch `internal/shell`.
- `PATTERN-spawn-observability`: the `source-file` invocation goes through reed's existing tmux call path, which already logs its spawns.
- `PATTERN-test-speed` and `PATTERN-test-economy`: the path rules are unit-tier; the sourcing behavior needs a real tmux and is `tmux`-tier.
- Docs land in the same commit: `internal/reedengine/doc.go` "Operator config" paragraph, both templates' comments, and `pattern/PATTERN-test-isolation.md`.
- Test budget: a new top-level test that pushes a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- The operator wants an opt-in key, not a reverted default; reed-hygiene chose hermetic servers one day before this task was filed, and the brief asks for a replacement key, not a revert.
- The operator's tmux config is a single file; a config that sources others does so itself through tmux's own `source-file`.
- One file per hub server is enough; nobody needs different tmux configs per worktree on one server, which tmux could not honor anyway.
- No `~/.tmux.conf` exists on the machine this run explored, so the live check by hand happens on the operator's own setup.

## Open risks

- An operator file that sets options reed relies on but does not pin (beyond D5's set) can still break a hub; D5 covers the two known server-killing options, and anything else surfaces as a bug report against that option.
- A file that runs plugins (tpm, status scripts) adds background CPU to every hub server; reed's two-line status bar replaces the operator's `status-format`, but `run-shell` and `#()` jobs in other options still run.
- A file that creates sessions puts extra sessions on the hub server; reed's discovery and reaping key on reed's own session names, and the plan writer confirms an unknown session is ignored rather than reaped or reported as an error.
- tmux's `~` and glob handling inside `source-file` is moot because reed passes an already-expanded absolute path; a path containing glob characters would still be globbed by tmux.

## Acceptance criteria

- With `tmux_conf` empty, a hub server reed starts carries none of a marker the operator's `~/.tmux.conf` would set (the existing hermetic behavior holds).
- With `tmux_conf` naming a file that sets a user option marker, a changed `prefix` and a changed `escape-time`, a hub server reed starts carries all three.
- With that file also setting `mouse`, `remain-on-exit`, `default-command`, `exit-unattached on` and `destroy-unattached on`, reed's values win on the booted server and session.
- A second worktree booting into the running server does not re-source the file.
- A file with a syntax error: the boot succeeds, the valid lines are applied, and a warning naming the path appears on the booting command's stderr and in the log.
- A relative path, a missing file and a directory are each refused at boot, naming `tmux_conf`, `LYX_REED_TMUX_CONF` and the path; `~/x` expands to the home directory.
- The capability probe and the server spawn argv still carry `-f /dev/null` on POSIX.
- `LYX_REED_TMUX_CONF` set in the environment of a tagged test package does not reach any server reed or the kit starts.
- On Windows a non-empty `tmux_conf` sources nothing.
- `lyx config reconcile` adds the key to a `reed.yaml` lacking it.

## Notes for the plan writer

- The boot path that starts a server and the pin order are in `internal/reedengine/lifecycle.go` (`serverSpawnArgv`, `pinBootOptionsLocked`); the `shell` refusal lives near the boot pre-flight referenced from `zoom.go`'s `withBootOpLockKeepingZoom` comment.
- The probe's `-f /dev/null` is in `internal/reedengine/probe.go`; it stays as is.
- `tmuxkit.Main` is in `internal/testkit/tmuxkit/tmuxkit.go`.
- A `tmux`-tier test of a server reed starts itself registers its key after reed's first boot, per `PATTERN-test-isolation`, since a kit pre-started server would bypass the spawn path.
