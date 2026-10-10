# Support log — reed-hygiene

## Interview

Autonomous run: no operator answered, so each batch below is the question the producer would have asked, with the pick it made.

1. Scope.
   Is this one task or three?
   One task: the board entry lists three fixes under one slug, each small, all in reed or its log, and the issue #495 covers the first two.
   Picked one task closing #495.
2. Daemon log.
   Where should the Debug lines go: a new daemon log file, the durable trace at a configurable level, or the durable trace when verbosity is Debug?
   Picked the verbosity switch (D1) and dropped the verb's stderr discard (D2), since the lines already exist and only the sink's level gate stops them; a config key is `logger-file-level`'s note.
3. Socket path.
   Refuse, or choose a short directory?
   Picked refuse (D6), since a reed-chosen directory needs `-S` on every client and the attach chain.
4. Server config.
   A reed-written config file like the kit's, or `-f /dev/null` plus session pins?
   Picked the pins (D7): per-worktree shell, no file, and tmux's `default-shell` wants an absolute path, which `exec.LookPath` gives.
5. Watchdog children.
   Code reading points at them before any measurement, as a hypothesis the re-measurement confirms or refutes: the per-session resize watchers in poll mode, through the worktree's tmux, which the shim did not wrap.
   Three decisions follow: the told tmux (D3), the signal entry at boot (D4), poll backoff (D5), then a re-measurement (D8).
6. Windows.
   Change it alongside or leave it?
   Left it, with bounds, since psmux's config and hook support is unverified and no host runs here.
7. Testing.
   Untagged fake-tmux tests for argv shapes and the pure checks; one `tmux`-tier boot under a marker `~/.tmux.conf`; a logger test for the Debug admission; the measurement as a manual card.

## Rejected alternatives

- A separate daemon log file under the hub logs directory: a new artifact with its own retention, when the durable trace already lives there.
- A `level` key in `logger.yaml`: the `logger-file-level` note's scope; it adds a config module change to every lyx process for a daemon-only need.
- Forwarding `-vv` from the spawner to the detached daemon: the inherited environment already carries `LYX_LOG_LEVEL`, and a flag would add a spawn argument for one diagnosis path.
- A reed-chosen short socket directory: needs `-S` in every client, the attach chain and the key bindings, and hides the server from the operator's own `tmux -L`.
- A reed-written tmux config file passed with `-f`: `default-command` is a session option, so pins give per-worktree shells with nothing on disk; the kit keeps its file for the two global lines reed never sets.
- Global `-g` shell options at boot: the first booter's shell would bind every sibling's strand panes.
- A lower poll ceiling for Windows resizes: the pattern's rule is one ceiling the loops share, and Windows is poll-only by an unverified capability, not by design.
- Passing `-f` on Windows too: unverified on psmux, and a wrong flag there costs the whole boot.
- Keeping the watchdog verb's stderr discard only for a detached daemon, keyed on a non-terminal stderr or a spawn flag: dropping `LYX_LOG_FILE` from the spawn's environment gives the same bound with no new flag and no terminal probe.
- A warning when a worktree's `tmux` value differs from the daemon's told path: a config value and a resolved path to the same binary differ as text, so it would fire on healthy hubs.
- One 107-byte socket limit for all of POSIX: macOS and the BSDs have a 104-byte `sun_path`, so a 104-to-107-byte path would pass the check and still fail in tmux.
- Measuring only sessions with strands: the earlier rows measured strand-less sessions, and the third row must compare.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): enabling switch for Debug in the durable sink. Picked process verbosity; rationale in D1 and A1.
- Q2 (auto-pick): keep or drop the watchdog verb's stderr discard. Dropped; rationale in D2.
- Q3 (auto-pick): which tmux binary the daemon's engines use. The told one; rationale in D3.
- Q4 (auto-pick): where the signal entry is first installed. At the fresh-boot pins; rationale in D4.
- Q5 (auto-pick): poll-mode ceiling. 60 seconds; rationale in D5 and A3.
- Q6 (auto-pick): refuse or relocate an over-long socket path. Refuse; rationale in D6.
- Q7 (auto-pick): config file or session pins for the shell options. Pins; rationale in D7 and A4.
- Q8 (auto-pick): Windows changes. None; rationale in A2.
- Q9 (auto-pick): re-measurement shape. Three idle strand-less sessions, as before; rationale in A5.
- Q10 (hypothesis from reading): what the watchdog's children are. The per-session poll-mode round trips, to be confirmed by the re-measurement; see D8.
- Environment: nothing outside the two discussion files looked broken during this session.
