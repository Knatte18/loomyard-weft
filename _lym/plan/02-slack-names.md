# Card 2 — slack-names

**Create:**
- `internal/lymdirs#SlackDirName`
- `internal/runproc#RoleSlackRelay`
- `internal/githubclient#CredentialDir`

**Edit:**
- `internal/lymdirs/dirs.go`
- `internal/runproc/doc.go`
- `internal/githubclient/doc.go`
- `internal/githubclient#TestCacheDirRedirection_HonoursOverride`

**Uses:**
- `internal/lymdirs/doc.go`
- `internal/githubclient#cacheDir`


**Intent:** Declare the three names the Slack relay's geometry and process record need, each once where its kind of name already lives.

- `lymdirs.SlackDirName` is the directory under the hub's board-level `.lym` (`fabricengine.HubScratchDir`) that holds the hub's Slack setup file, presence, cursor, outbound record, relay state, lock and log (D2, D3, D6), with a doc comment in the style of `CreelDirName`; the file header of `internal/lymdirs/dirs.go`, which lists the tokens the file declares, names the Slack directory too.
- `runproc.RoleSlackRelay` is the role the relay registers itself under `runproc.HubOwner` (D3); `internal/runproc/doc.go`'s list of roles names it.
  Because the relay is a `HubOwner` record and not a push or a sync, a prime anchor change's `EndDobbies` ends it, which card 13 restarts.
- `githubclient.CredentialDir` returns `cacheDir()`: the per-user lym directory the GitHub token cache lives in, where the Slack token file goes beside it (D2).
  `internal/githubclient/doc.go` says the package also names that directory for the Slack credential file, which `internal/slackengine` alone reads and writes; it resolves no Slack token itself.
  `TestCacheDirRedirection_HonoursOverride` gains one assertion that `CredentialDir()` answers the same directory and `ok` as `cacheDir()` under the redirected environment, so the export is pinned to the cache's directory.

**ImpactSummary:** Three additive declarations, one file header and two doc edits; no existing caller changes, and only the one githubclient test gains an assertion.
