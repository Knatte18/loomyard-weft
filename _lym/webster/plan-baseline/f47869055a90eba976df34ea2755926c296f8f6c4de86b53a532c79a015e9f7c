# Card 13 — Record the limit hit and switch the worktree's account

**Create:**
- `plan:internal/shuttleengine#AccountSwitcher` -> `type AccountSwitcher interface`
- `plan:internal/shuttleengine/claudeengine#Claude.SwitchAccount` -> `func (c *Claude) SwitchAccount(runDir string, turnEnd shuttleengine.Event) (bool, error)`
- `plan:internal/shuttleengine/claudeengine#usageLimitFallback` -> `const usageLimitFallback = 30 * time.Minute`
- `plan:internal/shuttleengine#Runner.SwitchAccountOnLimit` -> `func (r *Runner) SwitchAccountOnLimit(guid string, turnEnd Event) (bool, error)`
- `plan:internal/shuttleengine/claudeengine#TestClaude_SwitchAccount` -> `func TestClaude_SwitchAccount(t *testing.T)`
- `plan:internal/shuttleengine#TestRunner_SwitchAccountOnLimit` -> `func TestRunner_SwitchAccountOnLimit(t *testing.T)`

**Extends:**
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine#UsageLimitReader`
- `internal/shuttleengine#FindRun`
- `internal/shuttleengine/claudeengine#Claude.TurnEndActivity`
- `plan:internal/shuttleengine/claudeengine#readLaunchedAccount`
- `plan:internal/shuttleengine/claudeengine#recordBlocked`
- `plan:internal/shuttleengine/claudeengine#updateAccountFile`
- `plan:internal/shuttleengine/claudeengine#pickAccount`
- `plan:internal/shuttleengine/claudeengine#resolveAccount`
- `plan:internal/shuttleengine/claudeengine#hasHeadroom`
- `internal/shuttleengine#fakeEngine`

**Intent:**
D5's limit record and D7's switch decision, as one engine-neutral optional capability beside `UsageLimitReader` in `hold.go`: `AccountSwitcher.SwitchAccount(runDir string, turnEnd Event) (relaunch bool, err error)`.
`relaunch` true means the worktree now resolves to another account than the session's launched one, so the caller relaunches the session; false means the limit is handled as today.

`Claude.SwitchAccount` reads the launched record `account.json` in `runDir` (card 6); a session on an unlisted directory records nothing and returns false.
It records `blocked_until` on the launched account through `recordBlocked`: the reset `TurnEndActivity(turnEnd).UsageLimit.ResetsAt` names, or `usageLimitFallback` from now when the message names none, the later of it and a stored one winning.
With fewer than two accounts of positive weight it returns false.
Otherwise, in one `updateAccountFile` hold:
- under a hub pin it changes nothing and returns false, so the worktree waits the limit out as today;
- when the worktree now resolves (`resolveAccount`) to another account that has headroom, as after a sibling session's switch, it writes nothing and returns true;
- otherwise it runs `pickAccount` excluding the launched account with the fallback off, and on a taken account writes it as the worktree's entry with the counter, and returns true;
- with none eligible it changes nothing and returns false.
Under a beams pin the switch writes the limited worktree's entry only; the pinned account's `blocked_until` makes card 4's resolution take that entry until it passes, and the first launch after it returns to the pin.
A worktree with no entry on a listed directory switches to another account, never its own.

`Runner.SwitchAccountOnLimit(guid, turnEnd)` finds the run by its strand GUID and asks its engine; an engine without the capability answers false.
It is how the orch watcher (card 17) asks for the same switch; card 14's `Wait` and load hold ask their own engine directly.

Tests are tier0 over a temporary hub and fake account directories:
`TestClaude_SwitchAccount` rows: a hit with a stated reset records it as `blocked_until`, and one with none records 30 minutes on, an earlier hit never lowering a stored later one, and the account then has no headroom until it passes; another eligible candidate becomes the entry with its count incremented and the result is true; no other eligible account leaves the entry unchanged and returns false; a hub pin changes nothing and returns false; a beams-pinned worktree's entry is written while the pin's account is blocked, and resolution returns to the pin once `blocked_until` passes; an entry-less worktree on a listed directory switches to the other account; two calls for two sessions of one worktree at once make one switch, the second returning true with no write; an unlisted launched directory records nothing; with one account of positive weight it records the limit and returns false; after a switch, `Prepare` with the session as `ResumeSessionID` writes `account.json`, the status line and a `ResumeCmd` naming the new account, and `CheckResume` passes once `CarryTranscript` copied the transcript.
`TestRunner_SwitchAccountOnLimit` uses a `fakeEngine` wrapper implementing the capability: a known GUID reaches the engine with the run directory; an engine without it answers false; an unknown GUID is an error.

**ImpactSummary:** A new optional capability and a runner method; nothing calls them until cards 14 and 17.

**Commit:** 13: record a usage-limit hit on the launched account and switch the worktree's entry
