# Stage 1 of the closed area: supervisor, Landlock ruleset, connect filter, attach client and the in-area engines, token and relay

`area-seccomp-core` · Task · feature, reed, shuttle, claude, codex, creel, dobby, slack, process, cwd, git, cli, docs, pattern · draft

Decision record: [Sketch](decision-area-seccomp-core.md)

Depends on: [`area-creel-identity`](design-area-creel-identity.md), [`push-retry-not-halt`](design-push-retry-not-halt.md)

Filed from the Sketch `sandbox-closed-area-build`.

Takes: the creel identity part's exports (inside test, scope names, area registry writer, hub directory, area-aware level); config-keys-nested's rewritten PATTERN-config-layers line; docs/research/platform/seccomp-connect-probe/main.go as D4's starting point; reedengine's server spawn and scrub (lifecycle.go), reedcli's attach (EnsureSession, Status, the watchdog spawn), proc.CapProcessMemory and RunMemoryCapped, lymtemp's root and run directories, runproc's end verbs, claudeengine's account resolution, TranscriptReader and user-setup source, codexengine's homes and links, githubclient's token chain, slackengine's EnsureRelay, StopRelay and file geometry, creelcli's SendReady
Delivers: lym area init, start, stop, show, supervise, exec, git-credential and slack-setup; the area file outside the hub and its write-grant refusals; the supervisor in its own scope with its record, socket, SO_PEERCRED check, subreaping and verbs boot, attach-scope, cap-memory, kill-scope and slack-relay (start, stop, presence, status); the one launcher setting no_new_privs, Landlock ABI 8 with TSYNC and the filter with TSYNC, TSYNC_ESRCH and NEW_LISTENER; the connect decision on the checked inode and the derived allow list; the per-hub area root under lymtemp; the host-side attach client under the area; lym area start writing the machine record and the registry entry for an area hub; the host-side refusal of D7 as the first root pre-run step and at every hub selection, with machine walks skipping area hubs; dobby and creel spawn sites refusing from outside; the memory caps through the supervisor; the area's own engine config directories, GitHub token and relay records; README.md, docs/overview.md and the glossary per D16, the pseudo-terminal bound included (Added after Discussion); PATTERN-closed-area added, PATTERN-child-processes changed, and the backgrounds of PATTERN-child-processes, PATTERN-lym-files and PATTERN-config-layers per the notes for the plan writer; the seccomp half of the under-each-barrier criteria, whose apparmor half the fallback part carries; and the D6 attach criterion's second clause, which the record's unindented continuation line cuts from the parsed bullet: a `detach-client -E` sent to the attach client runs its command under the area's filter and in the attach scope
Exports: the supervisor's verb protocol and its client in internal/areaengine (one request per connection, the verbs named above), the launcher's start-request protocol, the area file type and its loader, the barrier enum seccomp and apparmor, and the start check's hook for a barrier-specific refusal, on which the fallback and host-exit parts add verbs and a barrier

The part changes these packages:

- internal/areaengine (about 3000 changed lines)
- internal/areacli (about 500 changed lines)
- cmd/lym (about 150 changed lines)
- internal/reedengine (about 250 changed lines)
- internal/reedcli (about 400 changed lines)
- internal/proc (about 250 changed lines)
- internal/lymtemp (about 120 changed lines)
- internal/runproc (about 40 changed lines)
- internal/dobbyengine (about 40 changed lines)
- internal/creelcli (about 80 changed lines)
- internal/machinecli (about 80 changed lines)
- internal/hubregistry (about 120 changed lines)
- internal/shuttleengine/claudeengine (about 250 changed lines)
- internal/shuttleengine/codexengine (about 200 changed lines)
- internal/githubclient (about 120 changed lines)
- internal/slackengine (about 350 changed lines)
- internal/slackcli (about 120 changed lines)
- internal/hubgeom (about 80 changed lines)
- README.md (about 80 changed lines)
- docs (about 80 changed lines)
- pattern (about 40 changed lines)
