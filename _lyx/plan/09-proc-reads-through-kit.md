# Card 9 — proc-reads-through-kit

**Edit:**
- `internal/loomcli#findWatchdogPIDs`
- `internal/loomcli#findDriverPIDs`
- `internal/orchcli#killTreeProcs`
- `internal/reedcli/smoke_proctree_test.go`
- `internal/pairteardown_test#processGone`
- `internal/gatecli#processGone`
- `internal/shedverbs#processEnded`
- `cmd/lyx#allowedTmuxkitOutsideTmuxTier`

**Delete:**
- `internal/loomcli#parentPID`

**Uses:**
- `internal/testkit/tmuxkit#ProcPPID`
- `internal/testkit/tmuxkit#ProcState`
- `internal/testkit/tmuxkit#ProcComm`
- `internal/testkit/tmuxkit#Pids`
- `internal/testkit/tmuxkit#ProcArgv`
- `internal/testkit/tmuxkit#ProcCwd`
- `internal/testkit/tmuxkit#IsWatchdog`

**Intent:** Make every test file outside `internal/testkit/tmuxkit` read `/proc` only through the kit's probes, decision D8, keeping each package's own matcher logic in its own test files and replacing only the copies whose logic is exactly a kit export.
`findWatchdogPIDs` iterates `tmuxkit.Pids`, reads argv through `tmuxkit.ProcArgv`, matches the adjacent `reed watchdog` pair through `tmuxkit.IsWatchdog` and keeps its own `--hub-path` match; its doc comment drops the `/proc`-native wording.
`findDriverPIDs` reads cwd through `tmuxkit.ProcCwd` and argv through `tmuxkit.ProcArgv`, keeps its `loom run` match and its parent-drop, and reads the parent pid through `tmuxkit.ProcPPID`, so `parentPID` loses its last caller and is deleted on this card.
`killTreeProcs` iterates `tmuxkit.Pids` and reads argv and cwd through the kit, keeping its dir-reference kill: an argv element that contains the dir, or a cwd at or under it.
`smoke_proctree_test.go` loses `linuxProcPPID` and `linuxProcComm`: `linuxDescendantClosure` builds its pid-to-ppid map from `tmuxkit.ProcPPID` and `linuxHubHolders` reads the comm through `tmuxkit.ProcComm`; its header comment no longer says the file reads `/proc` itself, and the file's `os`, `path/filepath`, `strconv` and `strings` imports shrink to what the remaining helpers use.
The three ended checks read the state letter through `tmuxkit.ProcState` and keep their fallbacks where the probe reports nothing: `pairteardown`'s `processGone` reports gone, and `gatecli`'s `processGone` and `shedverbs`'s `processEnded` fall back to `proc.IsAlive`; a `Z` state counts as ended in all three.
In `allowedTmuxkitOutsideTmuxTier`, the `internal/loomcli/smoke_helpers_test.go` row is reworded to name every probe the file now uses, and `internal/shedverbs/loop_integration_test.go` and `internal/gatecli/test_integration_test.go` each gain a row naming the state probe and that the file starts no server; `orchcli`, `reedcli` and `pairteardown` need no row, since their files are `llm`, `tmux || llm` and `tmux` tier.
Behavior is unchanged in every caller: the same processes are found, killed or reported ended as before.

**ImpactSummary:** Seven test helpers in six packages switch their reads to the kit; the only deletion is loomcli's parentPID, whose one caller is on this card, and no production code changes.

**Verify:** [ -z "$(grep -rl --include='*_test.go' '"/proc' internal cmd | grep -v '^internal/testkit/tmuxkit/')" ]
