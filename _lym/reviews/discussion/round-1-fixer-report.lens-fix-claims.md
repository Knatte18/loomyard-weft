```yaml
lens: fix-claims
outcome: reported
```

Task: check, through the fix-claims lens, that each claim the fix added or rewrote holds against the code at HEAD and the measurements.

**MEDIUM: D10 claims the `.in_use` files are written at session start, which the evidence contradicts**
- Changed text: D10 says "each written at that session's start". The support log's Round 1 calls them "the `.in_use/<pid>` files Claude Code writes" and, like D10, ties them to session start.
- Evidence: the marker for pid 3335 is dated Nov 26 08:43, while `ps` shows session 3335 started on Nov 23 at 20:21. For pid 1777338 the dates do agree: the process started Nov 26 08:22:36 and the marker is dated 08:22. So Claude Code also writes or rewrites these files after a session starts.
- The conclusion still holds: Claude Code writes under the plugin cache, so a read-only grant would fail. Only the timing is wrong; the line should say the file is written during the session, at its start and later.

**LOW: D8 says the pidfd stops a reused pid from standing for the sender, but no step reads through the pidfd**
- Changed text: D8 says "The pidfd pins the sender, so a pid reused after the sender exits never stands for it."
- Evidence: D3's inside test reads `/proc/<pid>/status` and the ancestry by pid. D8 never says the daemon checks the pidfd after those reads, for example by polling it or reading its fdinfo `Pid:` line.
- Without that step, a pid read alongside the pidfd can still name a reused process, so the guarantee goes further than the mechanism the record describes. The record should name that liveness check.

**Claims checked and found to hold**
- `serveHubCall` answers deliver and confirm before any sender is resolved (`internal/creelengine/receive.go:87`).
- `hubsToSweep` exists (`internal/machinecli/hubs.go:77`).
- The relay registers itself under `runproc.HubOwner`, and the `PATTERN-child-processes` background says it sits there (line 40).
- The scrub, `stripTraceID` and `stripIdentityEnv` are applied in `lifecycle.go`.
- `scopeStartScript` writes the cgroup marker.
- `lym slack setup` takes the `--operator` and `--channel` flags.
- codexengine's home links `auth.json` and `sessions` to the operator's home, per its `doc.go`.
- x/sys has no ABI-8 Landlock right that governs a Unix connect; `MAKE_SOCK` covers bind only.
- The watchdog's other spawn sites (`reed up`, `resume`, loom start) are host verbs that D7 refuses, so "starts only inside the area" holds.
- The Constraints' measurement lines match the probe output.

Tool calls:
- grep for `serveHubCall`, `hubsToSweep`, the watchdog spawn sites, `PATTERN-child-processes` and the Landlock constants: my context did not show those code sites or constants.
- grep to locate `serveHubCall` and check whether it reads any peer: my first grep missed the function body.
- `ps` with start times for the session pids: my context had the marker mtimes but not when those sessions started.
