# reed-tmux-conf — decision record

## Goal

Give the operator a way to put their own tmux configuration (prefix, key bindings, `escape-time` and the rest) back on reed's per-hub tmux servers, which since reed-hygiene (c6dc1fffc) start with `-f /dev/null` and read no `~/.tmux.conf`.
Reed's servers stay hermetic by default; the operator opts in by naming a file.

## Scope

In:

- A new `reed.yaml` key, `tmux_conf`, naming a tmux config file that reed sources into a hub server it has just started.
- Path validation for that key at boot.
- The ordering of that source against reed's own option pins, so reed's pins win.
- Identifying the one boot that sources the file, including under concurrent boots.
- Guarding reed's session and server lifetime against the file, and keeping sessions the file creates from holding the server.
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
Bound: this adds an edge through which arbitrary operator tmux commands reach reed's server; D3, D4, D5 and D8 bound what it can override, when it runs and what it can keep alive.

### D2 — Path rules: `~/` expanded, absolute required, file must exist, no glob or command-parser metacharacters

Decision: reed expands a leading `~/` to the user's home directory, then requires an absolute path to an existing regular file, containing no whitespace and none of `*`, `?`, `[`, `\`, `'`, `"`, `;`, `#`, `{`, `}` and `$`.
A relative path, a missing file, a non-regular file or a path with one of those characters is refused when `lyx reed up` (or any booting op) validates its config, at the same point and in the same way an unresolvable `shell` is refused today, naming the `tmux_conf` key, the `LYX_REED_TMUX_CONF` variable and the path.
The check runs whenever the key is non-empty, on every POSIX boot, whether or not that boot starts the server.
Rationale: a named file that does not exist is a configuration error, like an unresolvable `shell`, and a relative path has no meaningful base for a server whose cwd is the logs directory.
`source-file` treats its argument as a glob pattern, so an existing file whose name holds `[` or `\` is not found, and D4 would reduce that to a warning while the file silently never loads; `*` and `?` are refused with them because they make the path a pattern rather than one file.
D3's claim nests the path inside a command string that tmux parses a second time, so whitespace splits it into two paths, and quotes, `;`, `#`, braces and `$` change how it parses; each of these too would load nothing behind D4's warning.
Refusing them is one check with a clear error, where quoting for tmux's parser would have to track its rules exactly.
Checking on every boot keeps validation in one place instead of splitting it on whether a server happens to be running.

### D3 — Sourced once, at the server's start, before reed's pins

Decision: on POSIX, the boot that starts the hub server runs `source-file <path>` against that server after its session is up, at the head of D5's guard invocation and before `pinBootOptionsLocked` and the geometry pins.
The capability probe keeps `-f /dev/null` unchanged, and so does the server spawn, whose argv gains only D5's chained session guard.
`serverSpawnArgv` runs whenever a worktree's session is missing, so it does not identify the server's starter by itself; reed identifies it in two steps:

1. Before its spawn, a boot reads the hub server's `#{pid}` and `#{start_time}`, recording none when no server answers.
   Once its session is up, it reads them again, and the boot is a candidate only when the two reads differ.
   A boot whose spawn joined the server it saw is a join and sources nothing; a boot whose seen server ended before the spawn, so that the spawn started a new one, is a candidate.
2. A candidate's guard invocation opens with a claim: `if-shell -F` on the server-scoped user option `@lyx_tmux_conf_claimed`, whose unset branch sets it, to `sourced` when `tmux_conf` is non-empty and `empty` otherwise, and then runs `source-file <path>`, and whose set branch does nothing.
   A candidate whose `tmux_conf` is empty claims the same way and sources nothing, so the claim goes to the server's first starter whenever that starter reaches its guard invocation.

When two worktrees boot concurrently into a fresh hub, both can be candidates and both spawn, but exactly one claim wins: tmux runs a client's command queue without interleaving another client's commands until a command in it waits, the claim's check-and-set waits on nothing, and the set precedes the source; six concurrent claims against one tmux 3.6 server sourced the file once.
The losing candidate sources nothing, like a join.
Nothing re-sources the file into a running server.
Rationale: sourcing through a separate command keeps the probe's transient server and the spawn hermetic, so a config that creates a session cannot keep a transient server alive (the reason reed-hygiene gave the probe `-f /dev/null`).
Sourcing before the pins means every option reed sets, `-g` or per session, overwrites a conflicting value from the file.
`source-file` returns its errors to the caller, where `-f` at spawn would show them in a view-mode pane on the first session.
Reed's op lock is per worktree, so concurrent boots are not serialized, and only a check-and-set inside the server makes the starter unique.
A liveness check before the spawn alone would misclass a boot whose seen server is killed by another worktree's `down` before the spawn, leaving the new server unclaimed and its file unsourced until its next start; comparing the server before and after the spawn reads the server the spawn produced.
`start_time` beside `pid` keeps a reused pid from passing for the same server.
A server started before this change carries no claim, but every boot into it reads the same `pid` and `start_time` on both sides of its spawn and so is never a candidate.
Bound: the file can change only what reed does not pin.
A key binding, hook, server option or session-level default reed leaves untouched stays as the file sets it; this is the point of the feature, and the operator owns its consequences.
A value change takes effect at the hub server's next start, the same rule `debug_log` and `mouse` follow.
In a hub, the worktree whose boot wins the claim decides which file is sourced; another worktree's value is ignored, as its `tmux` value already is.
A server whose starter's boot ends between its spawn and its guard invocation, by an error or an interrupt, carries no claim, and no later boot is a candidate for it, so it runs without the file until its next start, which the last worktree's `down` brings about.

### D4 — Sourcing errors warn and do not fail the boot

Decision: when a guard invocation that ran `source-file` exits non-zero, reed logs a Warn record carrying the path and tmux's stderr, prints the same warning to the booting command's stderr, and continues the boot.
That invocation also carries D5's pins under one exit status, so its status never decides a pin failure; D5 step 3's read-back does.
Rationale: tmux parses the whole file before running any of it.
A parse error, such as an unclosed brace, applies nothing from the file, and the server runs on reed's own settings alone.
A command that fails at run time, such as an unknown option, is reported while every other command in the file is applied.
Either way the server is usable, and a typo in a personal config must not take a hub down.
The geometry pins already follow the same non-fatal rule.
Bound: only the guard invocation's exit status is non-fatal; the path validation in D2 still refuses, and D5 step 3's check still fails the boot.

### D5 — Reed guards the server and session lifetime against the file

Decision: on every POSIX boot that spawns a session, reed guards that session's lifetime in three steps, in this order:

1. The spawn invocation chains, after `new-session`, a session-local `destroy-unattached off` and D8's `@lyx_reed_session` tag on the new session.
2. Once the session is up, reed sends one guard invocation: D3's claim when the boot is a candidate, then `set -g destroy-unattached off` and the server option `set -s exit-unattached off`.
   Its exit status is D4's warning, never a boot failure.
3. In a separate invocation after that, reed checks that its own session still exists and reads back the global `destroy-unattached` and the server's `exit-unattached`.
   If the session is gone, the boot fails, naming `tmux_conf`, `LYX_REED_TMUX_CONF` and the path, and telling the operator to remove from the file the command that ends the session, such as an untargeted `set destroy-unattached on` or a `kill-session`.
   If either option reads other than `off`, the boot fails like a failed `remain-on-exit` pin.

A boot that is not a candidate sends the same guard invocation without the claim.
The pin set is closed at these two options.
Rationale: tmux runs a client's command queue without interleaving another client's commands until a command in it waits.
A waiting command in the operator file, such as `run-shell` without `-b` (tpm's last line), lets other clients run between the file's lines and reed's pins and see the file's values; step 1's chaining and the non-waiting claim keep that window harmless, and the guard client stays connected through the wait, so the server outlives the file's `exit-unattached on`.
A detached session dies the moment `destroy-unattached on` applies to it, before any later pin can run, and under concurrent boots another worktree's claim can apply the file's global value right after this boot's `new-session`; chaining the session-local `off` into the spawn guards the session from its creation, and it outranks the file's `set -g destroy-unattached on`, the form a tmux config uses.
`exit-unattached on` ends the server as soon as its last client disconnects, and each reed tmux call is its own client, so on tmux 3.6 a pin in a separate invocation after the source never reaches the server; pinning in the claim's own invocation keeps one client connected from the source through the pin.
On tmux 3.6 a file with a run-time or parse error makes that invocation exit 1 while both pins still apply, so the exit status cannot tell a source error from a pin failure, and the read-back can.
The `-g` pin keeps later worktrees' sessions from inheriting the file's global value.
Both options default to `off`, so with an empty `tmux_conf` the steps leave the server as it is today, and the steps run on every boot so that there is one boot path.
Bound: a file that ends the boot's session by any route fails that boot loudly instead of leaving a half-booted hub.
An option reed does not pin that an operator file flips is the operator's to own, per D3's bound.

### D6 — Test servers never source a developer's config

Decision: `tmuxkit.Main` unsets `LYX_REED_TMUX_CONF` in the test process's environment, beside the `TMUX_TMPDIR` and `TMPDIR` it already sets, so a `reed.yaml` rendered from the template resolves `tmux_conf` to empty in every tagged test package.
A test that exercises the feature sets the key explicitly to a file it wrote.
Rationale: `PATTERN-test-isolation` requires tests to reach no operator global state, and the `${env:...}` default makes the developer's shell environment an input otherwise.

### D7 — Windows ignores the key

Decision: on Windows the key is parsed but neither validated nor sourced, and the Windows template comment says so.
Rationale: reed-hygiene left Windows boot unchanged, psmux reads its own config at start, and its `source-file` support is unverified.

### D8 — Sessions an operator file creates never keep the hub server alive

Decision: on POSIX, reed tags each session it creates with the session user option `@lyx_reed_session`, chained into the spawn invocation by D5 step 1, so a session carries the tag from the moment it exists.
On a server whose `@lyx_tmux_conf_claimed` is `sourced`, a session counts as reed's only when it carries the tag; on any other server every session counts, as today.
Every reed read that lists, counts or cycles the server's sessions sees only the sessions that count as reed's, whether it runs in Go or in a tmux format, such as the status bar's session row.
So the teardown paths, which kill the server when no reed session remains, end the server with the last reed session and a file-created session with it, and no reed surface attaches to, reaps, reports, shows or switches to a session that does not count; it lives until the server ends.
Rationale: teardown kills the server only on an empty `list-sessions`, so a file-created session would keep the server running after the last worktree goes down, every later boot would join it, and a changed file would never reach a server, against D3's next-start rule.
A rule over every session read, rather than a list of surfaces, keeps a surface added later from showing the file's sessions by default.
Tagging in the spawn invocation leaves no window in which a booting session goes uncounted, so a concurrent last-worktree `down` still sees it and leaves the server up.
Limiting the tag rule to servers where a file was sourced keeps a server started before this change, whose reed sessions carry no tag, on today's rule.
Bound: the tag rule lets the server be killed while a file-created session lives and hides that session from reed, and nothing else; a reed session still keeps the server alive.

### Added after Discussion (parent, 2026-10-11): Pin failure is read back, not inferred from the guard's exit status

Decision: The guard invocation (claim, source-file, pins) exits non-zero whenever the operator file has a parse or run-time error, so its non-zero exit is D4's warning, never fatal. Pin failure is detected separately: the step-3 check invocation reads back global destroy-unattached and exit-unattached and fails the boot if either is not off. D4's trigger becomes 'the guard invocation exits non-zero' rather than 'source-file exits non-zero'. Add an acceptance criterion: a file with a run-time error that also sets 'set -g exit-unattached on' boots, warns, and leaves both pins off.

Rationale: Round 3 F1, probed on tmux 3.6: one tmux exit status carries the claim, the source and both pins, and a typo in the operator file exits 1 while both pins applied. Reading the pins back is the only signal that separates the two, and it keeps D4's rule that a typo in a personal config must not take a hub down.

### Added after Discussion (parent, 2026-10-11): Client queues interleave at a waiting command

Decision: D3 and D5 state the premise as: tmux runs a client's queue without interleaving other clients until a command waits; a waiting command in the operator file (run-shell without -b, as tpm's last line) lets other clients run between the file's lines and reed's pins. D5 step 1's chained session-local destroy-unattached off with the tag, and the non-waiting claim, are what keep that window harmless; the plan must not treat the file's effect as atomic against other boots.

Rationale: Round 3 F2, probed on tmux 3.6: during a sourced run-shell 'sleep 2' another client saw the file's global on values and an unchained new-session died, while the chained one survived.

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

## Open risks

- An operator file that sets options reed relies on but does not pin can still break a hub; D5's closed pin set covers the two options that end the server or session, and anything else surfaces as a bug report against that option.
- A file that runs plugins (tpm, status scripts) adds background CPU to every hub server; reed's two-line status bar replaces the operator's `status-format`, but `run-shell` and `#()` jobs in other options still run.
- An untargeted session-level command in the file applies to whichever session tmux takes as current, which under a concurrent boot can be another worktree's session; D5's chained guard covers `destroy-unattached` on every reed session, and its session check covers only the sourcing boot's own session against anything else.

## Acceptance criteria

- With `tmux_conf` empty, a hub server reed starts carries none of a marker the operator's `~/.tmux.conf` would set (the existing hermetic behavior holds).
- With `tmux_conf` naming a file that sets a user option marker, a changed `prefix` and a changed `escape-time`, a hub server reed starts carries all three.
- With that file also setting `mouse`, `remain-on-exit`, `default-command`, `set -g exit-unattached on` and `set -g destroy-unattached on`, the boot succeeds and reed's values win on the booted server and session, and a second worktree's session booted later into that server keeps `destroy-unattached off`.
- A file with an untargeted `set destroy-unattached on` fails the boot, naming `tmux_conf`, `LYX_REED_TMUX_CONF` and the path.
- A second worktree booting into the running server does not re-source the file.
- Two worktrees booting concurrently into a fresh hub source the file exactly once, which a file appending to a user option shows.
- Two worktrees booting concurrently into a fresh hub with a file setting `set -g destroy-unattached on` and `set -g exit-unattached on` both succeed, and both sessions keep `destroy-unattached off`.
- A boot whose pre-spawn read saw a server that has ended by its spawn sources the file into the server its spawn starts.
- A file with a parse error: the boot succeeds, nothing from the file is applied, and a warning naming the path appears on the booting command's stderr and in the log.
- A file with a run-time error, such as an unknown option between two valid lines: the boot succeeds, both valid lines are applied, and the same warning appears.
- A file with a run-time error that also sets `set -g exit-unattached on` and `set -g destroy-unattached on`: the boot succeeds, the same warning appears, and the server's `exit-unattached` and the global `destroy-unattached` read `off`.
- A file that creates a session: after every worktree's `down`, the hub server is gone, and while it lives the leftover session is never reported by reed's status or discovery, shown in the status bar's session row, or reached by the next and previous session switch.
- A relative path, a missing file, a directory, and existing files whose names contain `[` and a space are each refused at boot, naming `tmux_conf`, `LYX_REED_TMUX_CONF` and the path; `~/x` expands to the home directory.
- The capability probe and the server spawn argv still carry `-f /dev/null` on POSIX.
- `LYX_REED_TMUX_CONF` set in the environment of a tagged test package does not reach any server reed or the kit starts.
- On Windows a non-empty `tmux_conf` sources nothing.
- `lyx config reconcile` adds the key to a `reed.yaml` lacking it.

## Notes for the plan writer

- The boot path that starts a server and the pin order are in `internal/reedengine/lifecycle.go` (`serverSpawnArgv`, `pinBootOptionsLocked`); the `shell` refusal lives near the boot pre-flight referenced from `zoom.go`'s `withBootOpLockKeepingZoom` comment.
- The probe's `-f /dev/null` is in `internal/reedengine/probe.go`; it stays as is.
- `tmuxkit.Main` is in `internal/testkit/tmuxkit/tmuxkit.go`.
- A `tmux`-tier test of a server reed starts itself registers its key after reed's first boot, per `PATTERN-test-isolation`, since a kit pre-started server would bypass the spawn path.
