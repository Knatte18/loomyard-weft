# Card 1 — hub-registry

**Create:**
- `internal/standalonestate#UserStateDir`
- `internal/standalonestate#stateRootFor`
- `internal/standalonestate#TestUserStateDir`
- `internal/hubregistry#RecordFileName`
- `internal/hubregistry#Entry`
- `internal/hubregistry#Record`
- `internal/hubregistry#Lookup`
- `internal/hubregistry#Live`
- `internal/hubregistry_test#TestRecord`
- `internal/hubregistry_test#TestLookup`
- `internal/hubregistry#ErrUnresolved`
- `internal/hubregistry/doc.go`
- `internal/hubregistry/hubregistry_test.go`
- `internal/hubregistry/testmain_test.go`

**Edit:**
- `internal/standalonestate#stateDirFor`
- `internal/testkit/tmuxkit#setTempEnv`
- `internal/testkit/tmuxkit#TestSetEnv`

**Extends:**
- `internal/standalonestate/doc.go`
- `docs/overview.md`
- `pattern/PATTERN-test-isolation.md`

**Uses:**
- `internal/fabricengine#ReadShortname`
- `internal/fabricengine#BoltDir`
- `internal/state#UpdateJSON`
- `internal/state#ReadJSON`

**Intent:** Add the machine record of hubs (D1), which no later card can do without.

`internal/standalonestate`: split `stateDirFor` into `stateRootFor`, the per-OS `lym` directory under `$XDG_STATE_HOME` or `~/.local/state` (under `%LOCALAPPDATA%` on Windows), and the hash8 join, with every existing refusal unchanged.
`UserStateDir` reads the same environment `Derive` reads and answers `stateRootFor`, creating nothing.
It adds no production caller of `Derive`, so `internal/cliwire` stays that function's only one, and the package stays stdlib-only.
`TestUserStateDir` sets `XDG_STATE_HOME` to an absolute path, then a relative one that lands on the `HOME` fallback; it is tier0, and it touches process-global environment, so it is not parallel and says so.

`internal/hubregistry` is told its directory on every call, so a test's record lives in that test's own directory.
The record is `RecordFileName` in `dir`: one JSON object holding a list of `Entry`, each a hub path, its shortname and the time it was last recorded.
Its name is the one constant `RecordFileName`, and its lock file is a second named constant beside it, in the same per-user directory, since the record belongs to no hub (the `internal/githubclient/cache.go` precedent).

- `Record` rewrites the file through `state.UpdateJSON`, which holds the `internal/lock` write lock across the read and the write, so two daemons recording at once lose no entry.
  In that write it upserts `hubPath` with `shortname` and `now`, and drops every other entry whose hub path no longer exists, or whose `fabricengine.ReadShortname(fabricengine.BoltDir(path))` answers found with a shortname other than the entry's.
  An entry whose record reads not found, and one whose hub path a stat answers with anything but not-exist, is kept.
- `Lookup` reads the file through `state.ReadJSON`, reads each entry's Bolt shortname record and keeps the entries that read `shortname`.
  With one it answers that hub path.
  With none it refuses: `creel: no hub with the shortname "<s>" is recorded on this machine; way forward: run lym creel inbox in that hub once`.
  With several it refuses, opening with that mail across hubs needs distinct shortnames, then the way forward of removing the extra clone, since a recorded shortname never changes, and then every hub path, so the 200-byte cut of D2's notice loses only paths.
  A file that is absent reads as no entries, and so reaches the none refusal.
  The none and several refusals wrap `ErrUnresolved`, so a caller tells them from a record that cannot be read.
- `Live` answers the entries whose Bolt shortname record reads the entry's own shortname, in shortname order.
- A record `state` cannot decode is refused by `Lookup`, `Live` and `Record` alike with the file's path and the way forward of removing the file by its path, after which `lym creel inbox` in each hub, a minute on, records it again (card 4).

`TestRecord` and `TestLookup` are in `internal/hubregistry/hubregistry_test.go`, which is tier0: its hubs are temporary directories holding only `_bolt/.lym-shortname`, so it spawns nothing.
`internal/hubregistry/testmain_test.go` is untagged and calls `tmuxkit.Main`.
`TestRecord` rows: a first record; a second record of one hub updates its time and leaves one entry; an entry whose path is gone is dropped; an entry whose path lies under a regular file, which a stat answers with no not-exist, is kept; an entry whose shortname record reads another valid shortname is dropped; an entry whose shortname record is absent is kept; many goroutines each recording a hub of its own at once all stay; a corrupt file is refused naming the path.
`TestLookup` rows: the one match; an unknown shortname refused naming `lym creel inbox` and matching `ErrUnresolved`; two hubs sharing a shortname refused naming both paths and matching `ErrUnresolved`; a hub whose shortname changed since it was recorded drops out; a removed hub drops out; an absent file; `Live` lists exactly the entries that pass; a corrupt file, which does not match `ErrUnresolved`.

`PATTERN-test-isolation` needs no test to write the operator's record, and a real `lym creel daemon` a test spawns through `lymbin` derives its directory from the environment.
So `setTempEnv` also sets `XDG_STATE_HOME` to the run directory, and on Windows `LOCALAPPDATA`, which every process a test starts inherits; its comment and `PATTERN-test-isolation`'s private temp directory section say so.
A test that sets the variable itself still wins.
`TestSetEnv` saves both variables through `t.Setenv` before `setEnv`, so it leaks neither, and its list of variables pointed at the directory gains `XDG_STATE_HOME`.

`docs/overview.md` gets a package-tree line for `internal/hubregistry/`.

**ImpactSummary:** One new leaf-side package with no caller yet; `stateDirFor` keeps its signature and its one caller, and every test binary gains a hermetic `XDG_STATE_HOME`, which the standalone tests that set it themselves override.
