```yaml
lens: fix-claims
outcome: reported
```

Task: check every claim the fix added or rewrote against the code at HEAD (the fix-claims lens).

no findings

What I checked against HEAD:
- **Card 5 call sites:** `start` (`internal/slackcli/cli.go:387`) and `away` (`cli.go:459`) call `RequireConfigured`. `setup` reads the token (`cli.go:304`) and refuses with `noTokenMessage` (`cli.go:309`). The `RunRelay` check sits before the lock (`relay.go:186`). `starter.ensure` reads the token (`start.go:172`).
- **Card 5 test sites:** the row `StoresTheTokenOnlyAfterSlackAcceptsIt` exists (`cli_test.go:368`). The `WriteToken` and `ReadToken` call sites are the ones the card names: `configure`, setup, login and `StartAndAwayNeedLoginAndSetup` in `TestSlackCLI`; slackengine `newHarness`; `TestRunRelay`; `TestEnsureRelay`; `TestCredentialFiles`; the beamcli test.
- **Card 6 `MissingScopeNamesTheScopes` claim:** `scopeRefusal` formats `botScopes` (`cli.go:360`), so the reworded scopes and `docs/slack-app.md` do reach the row's refusal.
- **Card 13 doc claims:** `OpenConnection` and `PostAs` are real planned symbols (`01-api-web-methods.md:15-16`), and the sentences to drop match `internal/slackengine/doc.go` lines 8, 11, 26, 29, 37 and 56-57.
- **Card 12 stale-line row:** it matches `reportStale` (`outbound.go:136-147`), which clears the count only after a post that stands.

Tool calls:
- One grep of `internal/slackcli/cli.go` for the verb functions, `noTokenMessage`, `RequireConfigured` and `ReadToken`, and of the plan cards for `OpenConnection` and `PostAs`. My context had neither the call sites of `start`, `away` and `setup` nor where cards 1–8 define those two symbols.
