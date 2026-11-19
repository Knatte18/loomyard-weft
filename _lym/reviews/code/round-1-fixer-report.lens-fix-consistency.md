```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens on the round-1 change set (look for twins left stale).

**Findings**

1. **LOW** — `internal/boardserve/doc.go:42` (twin of the F5 change in `server.go`). The package doc says the server ends when "no API request came within the idle bound". Since F5, only a request that carries a valid token counts as activity (`server.go:113`, `server.go:120`). The same loose wording remains in the `DefaultIdleBound` and `Options.IdleBound` comments (`run.go:29`, `run.go:84`), and in the `watchBoard` doc ("no API request came", `run.go:212`). Evidence: `stampAPIRequest` is called only after `bearerAuthorized` succeeds in `ServeHTTP`.

2. **LOW** — `internal/boardserve/doc.go:39-41` (twin of the F1 change in `run.go`). The Lifecycle section says the loop "wakes on a change event of board.json or on a timer" and does not mention the new one-second floor between event reads. `PATTERN-daemon-wakeups.md:23` and the `watchBoard` doc were updated, so the package doc is the one description that was missed.

3. **LOW** — `internal/boardserve/doc.go:10` (twin of the F2 change). The text reads "With after=&lt;revision&gt; equal to the current revision it waits". It does not say that a request with no `after` key answers at once, or that an empty `after` equals the current revision for a board with no `board.json`. The sentence is still true, but this is the only statement of the long-poll contract and it omits the rule F2 introduced.

No other twins found. The other `poll('')` call sites and `apiPathPrefix` references are gone. The 409 and 422 split in `doc.go:30` still agrees with F7 and F6.

**Calls made**
1. `grep` across `internal/boardserve`, `internal/boardengine`, `internal/boardcli`, `pattern` and `docs` for the changed behaviours. I needed to know whether other text states the old rules, and my context only held the edited files.
2. `Read` of `internal/boardserve/doc.go`. It was not in my context, and the grep showed it states the long-poll, idle and watch-loop contracts.
