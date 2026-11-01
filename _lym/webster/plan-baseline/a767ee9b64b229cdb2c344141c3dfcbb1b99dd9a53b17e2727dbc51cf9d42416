# Card 4 — await-resolved-cap

**Edit:**
- `//internal/shuttlecli/await.go`
- `//internal/shuttlecli/cli.go`
- `internal/shuttleengine#DefaultAwaitCap`
- `//internal/shuttleengine/doc.go`
- `//docs/overview.md`

**Create:**
- `internal/shuttlecli#awaitCap`
- `internal/shuttlecli#TestAwaitCap`

**Uses:**
- `internal/shuttleengines#KeepWarm`
- `internal/shuttleengines#KeepWarmAnswer`
- `internal/shuttlecli#TestAwaitVerb`

**Intent:** `lym shuttle await`'s `--cap` default reads the resolver (D4), and the text naming a keep-warm Bash timeout says 3600000 ms (D9).

`awaitCap(cwd, strand, flagCap, flagGiven, keepWarm, resolveWorktree)` answers the cap a call waits:
when `flagGiven` it answers `flagCap` and calls neither function, so an explicit `--cap` wins;
otherwise, when `resolveWorktree(cwd)` finds a git worktree it answers `keepWarm(fabricengine.ConfigDirs(location), strand).Cap`, logging at Debug when the answer is a fallback;
when it fails (no git worktree, the operator outside any repository) it answers `shuttleengine.DefaultAwaitCap` without calling `keepWarm`, logged at Debug, so the verb still needs no hub or repository.
`awaitCmd`'s RunE waits `awaitCap(cwd, os.Getenv(agentname.StrandNameEnv), <the flag's value>, cmd.Flags().Changed("cap"), shuttleengines.KeepWarm, lymcwd.ResolveWorktree)`; the flag's own default stays `DefaultAwaitCap` and its usage says it defaults to the caller role's keep-warm cap.
The help text says the default is the caller's role's keep-warm cap from claude.yaml inside a worktree and 250s elsewhere, and to run the command with a Bash tool timeout of 3600000 ms; the 300000 ms text goes.
`DefaultAwaitCap`'s doc comment says it is the fallback when the keep-warm resolver cannot answer, that a caller runs a keep-warm wait with a Bash tool timeout of 3600000 ms so a long or raised cap fits, and no longer says `lym gate test` waits for a slot for at most it.
The shuttleengine doc's sentence on `DefaultAwaitCap` (beside `AwaitAny`) says it is the fallback of `shuttleengines.KeepWarm`.
`await.go`'s file header no longer says the verb reads no file's content, and says it reads the keep-warm config inside a worktree;
`awaitVerb`'s comment says `Command`'s PersistentPreRunE still skips resolution for it, since the verb resolves only its own config dirs when its directory lies in a git worktree and needs no hub.
`cli.go`'s file header and `Command`'s comment, which say the await verb needs no config or engine, say the same: await skips the resolution and needs no hub, reading only the keep-warm config inside a git worktree.
`docs/overview.md`'s shuttle paragraph says `lym shuttle await`'s cap defaults to the caller role's keep-warm cap.

`TestAwaitCap`, tier0, injects both functions: a found worktree answers the fake resolver's cap (50ms) and hands it the worktree's `fabricengine.ConfigDirs` and the strand; a fake resolver answering a fallback still answers its cap; `lymcwd.ErrNotAGitRepo` answers `DefaultAwaitCap` and the fake resolver records no call; a given `--cap` of 30ms inside a found worktree whose fake resolver answers 50ms answers 30ms, and neither fake records a call.
`TestAwaitVerb`'s existing rows all pass `--cap` or meet a path at once, so they never wait out the resolved cap.

**ImpactSummary:** one verb's default cap and help text, and one constant's doc comment; the constant's value and every other caller stay.
