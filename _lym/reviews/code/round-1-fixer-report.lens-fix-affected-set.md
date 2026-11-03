```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the F1 and F6 set searches and check the fix covers every member.

Findings:

- **LOW** — F1, `internal/fabriccli/hibernatedaemon_test.go`. The test pins the ended-phase values but not the request file name.
  - The review's own proposal for F1 included "the request file name" in the shared source of truth.
  - `hibernateRequestFileName = "request.json"` (`internal/fabriccli/anchordeps.go:41`) is a second copy of `requestFileName` (`internal/hibernatecli/deps.go:35`).
  - The new `TestHibernateDaemonLive` writes the literal `"request.json"` itself. A rename in `hibernatecli` would leave `fabriccli` reading "no request" with every test green.
  - The grep for `"request.json"` found no other hibernate copy. The `gatecli` and `reportengine` hits are unrelated requests.
  - The proposal's wording is about phases, so this gap is in the set the proposal named, not an extra improvement.

No other findings:

- **F1 phase literals:** the only other site holding the terminal-phase literals outside `hibernateengine` is `fabriccli/anchordeps.go:45`. The `sheddriver/drive_test.go:77` hit is unrelated. The fix pins that one site, and `TestHibernateEndedPhases_MatchTheEnginesTerminalPhases` checks it both ways.
  - The test's phase list is hand-enumerated. A new engine phase left out of the list would not be checked, but the second loop still rejects any non-terminal entry in the ended list.
- **F6 "hibernate daemon" wording:** no other `.go` or `.md` file under `internal`, `cmd` or `docs` says "hibernate daemon" or "hibernation daemon". The remaining "daemon" uses (`DaemonPID`, "the daemon that carries it") are the engine's established term for the carrying dobby, not the removed spawn.
- **F3 hibernate directory:** `NewDobbyCarrier` is the only caller that wires the prime's watch to the request path (`beamcli/dobbydeps.go:194`). `hibernateDir` and `HibernateDirName` have no other non-test use that needs the directory to exist.

Tool calls:
1. `grep` for the terminal-phase literal combinations outside `hibernateengine`: my context held no search of the repo for repeated phase values.
2. `grep` for "hibernate daemon" wording across `.go` and `.md`: my context held only the single line the review named.
3. `grep` for `"request.json"` and `HibernateDirName` in non-test code: my context did not show every site that builds the request path.
