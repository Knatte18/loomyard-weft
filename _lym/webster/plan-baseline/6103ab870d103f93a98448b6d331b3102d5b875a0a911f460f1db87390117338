# Card 9 — `lym claude account set|show` and the status line verb

**Create:**
- `internal/claudecli/doc.go`
- `internal/claudecli/testmain_test.go`
- `internal/claudecli#Command`
- `internal/claudecli#accountKeeperHere`
- `internal/claudecli#TestCommand`
- `internal/shuttleengine/claudeengine#Claude.SetAccount`
- `internal/shuttleengine/claudeengine#Claude.ShowAccounts`
- `internal/shuttleengine/claudeengine#Claude.RecordStatusLine`
- `internal/shuttleengine/claudeengine#parseStatusLine`
- `internal/shuttleengine/claudeengine#TestClaude_RecordStatusLine`
- `internal/shuttleengine/claudeengine#TestClaude_SetAccount`

**Extends:**
- `internal/shuttleengine#AccountKeeper`
- `cmd/lym#newRoot`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengines#AccountKeeper`
- `internal/hubgeom#EngineWorktree`
- `internal/fabricengine#ConfigDirs`
- `internal/shuttleengine/claudeengine#writeUsage`
- `internal/shuttleengine/claudeengine#updateAccountFile`
- `internal/shuttleengine/claudeengine#resolveAccount`
- `internal/shuttleengine/claudeengine#hasHeadroom`

**Intent:**
`AccountKeeper` gains `SetAccount(name string) error`, `ShowAccounts() (string, error)` and `RecordStatusLine(name string, input []byte) (string, error)`.
A new `internal/claudecli` package holds the cobra subtree `lym claude`, added to the root in `newRoot` beside the others (`PATTERN-cli-cobra`); it reaches the engine only through `shuttleengines.AccountKeeper` (`PATTERN-shuttle-provider-seam`).
`accountKeeperHere` resolves the current worktree through `lymcwd`, and builds the keeper from `fabricengine.ConfigDirs` and `hubgeom.EngineWorktree` of it; outside a hub it refuses, naming that accounts belong to a hub and the way forward: run the verb from a worktree of a hub.
`internal/claudecli/testmain_test.go` is untagged: it holds `TestMain` only, which runs the package through `tmuxkit.Main` (`PATTERN-test-isolation`).

- `lym claude account set <account>` runs `SetAccount`: it writes the worktree's entry in one `updateAccountFile` hold, taking effect at the worktree's next engine-built launch; running strands keep their account (D7).
  It refuses an account the list lacks or has retired, naming the listed ones; under a hub pin it refuses, naming the pin and `lym config claude --layer machine`; under a beams pin it writes the entry, which takes effect while the pin is out.
  Bound: any process that can run lym from a worktree's directory, an agent's included, can set that worktree's entry; it changes only the machine-local entry, refuses an account the list lacks or has retired, and cannot override a hub pin or a beams pin that is not out.
- `lym claude account show` prints the worktree's resolved account (`resolveAccount`), the two pins, and each listed account's weight, windows with their stored percentage and reset, `blocked_until`, and whether it has headroom.
- `lym claude statusline <account>` runs `RecordStatusLine` over stdin: it refuses a name no listed account has and writes nothing; `parseStatusLine` reads `rate_limits.five_hour` and `rate_limits.seven_day`, each `used_percentage` and `resets_at`, and drops a window whose percentage lies outside 0 to 100 or whose reset lies more than its window's length (five hours, seven days) ahead; input without `rate_limits` writes nothing; the rest goes through `writeUsage` (card 4), which keeps `blocked_until`.
  It prints one short line: the account name and each window's percentage.
  Bound (D5): any shell that can run the verb can set both windows of every listed account, up to each window's length ahead, and can replace stored figures with a later reset; it never writes or clears `blocked_until`.

Tests are tier0 over a temporary hub and fake account directories:
`TestClaude_RecordStatusLine` rows: both windows written for a listed name; input without `rate_limits` writes nothing; an unlisted name is refused and writes nothing; a percentage of 101 and a reset eight days ahead are dropped and the other window written; the stored `blocked_until` survives; the printed line names the account and both percentages.
`TestClaude_SetAccount` rows: an entry is written and `resolveAccount` then returns it; an unlisted and a retired account are refused; a hub pin refuses, naming the pin; a beams pin writes the entry; `ShowAccounts` names the resolved account, both pins, and each account's weight, windows with percentage and reset, `blocked_until` and headroom.
`TestCommand` checks the subtree's verbs, their argument counts, and the refusal outside a hub, which runs under `t.Chdir` and so not in parallel.

**ImpactSummary:** A new CLI package and one root registration; three methods on the claude engine behind the existing capability; nothing else calls them.

**Commit:** 9: lym claude: account set and show, and the status line verb
