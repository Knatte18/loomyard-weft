# Relevant files

- `docs/research/loom/adopt-build-revision-2026-11-16/report.md` — the study of what landed and what each remaining part takes from it
- `docs/research/loom/adopt-build-revision-2026-11-16/record-push-before-act-takeover.md` — the revised synchronous push and takeover record D1 and D2 come from
- `docs/research/loom/adopt-build-revision-2026-11-16/record-bolt-holds-catch-up.md` — the revised registry, catch-up and holds record D3 to D5 come from
- `docs/research/loom/adopt-build-revision-2026-11-16/record-adopt-foreground-leftovers.md` — the revised foreground adopt record D6 comes from
- `docs/research/loom/adopt-build-revision-2026-11-16/record-leftover-pairs-teardown.md` — the revised leftover-pair record D7 comes from
- `internal/tendercli/runintents.go` — the pause and resume intents commit, then spawn the detached push D1 makes synchronous
- `internal/battencli/cancelcmd.go` — the cancel intent and finish commit, then spawn the detached push D1 makes synchronous
- `internal/loomcli/landingdeps.go` — the pull-request record's committer and the fixer counts' records directory
- `internal/tenderengine/resume.go` `spawnOwed` — the deferred pushes a lift pays, which D7 and D9 keep
- `internal/hibernateengine/watch.go` — the pause-and-confirm sequence D9 shares
- `internal/hubregistry/hubregistry.go` `Live` — the hubs D10 starts and the launchers it keeps
- `internal/termwindow/termwindow.go` `Resolve` — opens the terminals D10 needs, Linux and WSL only today
- `pattern/PATTERN-owner-machine.md` — describes the origin-owed marker and owner pause that D4 and D5 retire
- `pattern/PATTERN-dobby-runs.md` — its adopt pause cluster retires with D6
- `pattern/PATTERN-test-only-code.md` — places the takeover writer, hold setter and catch-up callers
