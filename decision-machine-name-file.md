## Goal

machinename reads the machine name from machine.json before the hostname rule

Takes: the decision record's D1-D4, D7 and D8 for internal/machinename's tests
Delivers: machinename.Name reading machine.json from the absolute per-user config directory before the hostname rule, with every name error naming its own way forward; githubclient.CredentialDir delegating to standalonestate.UserConfigDir; battencli.DobbyLandings counting landings under former names; the name-error wrappers in internal/battencli and internal/beamcli rid of their hostname fix
Exports: standalonestate.UserConfigDir, standalonestate.AbsoluteUserConfigDir, machinename.FilePath, machinename.ReadFile, machinename.Names, machinename.CheckName, battencli.DobbyLandings taking this machine's names

## Scope

The part changes these packages:

- internal/standalonestate (about 100 changed lines)
- internal/githubclient (about 15 changed lines)
- internal/machinename (about 180 changed lines)
- internal/battencli (about 30 changed lines)
- internal/beamcli (about 50 changed lines)

## Decisions

### D1 — The machine file lives in the per-user config directory

The file is `machine.json` in the per-user lym config directory: `$XDG_CONFIG_HOME/lym`, else `$HOME/.config/lym`, and `%LOCALAPPDATA%\lym` on Windows, resolved as D2 says.
It is the directory `githubclient.CredentialDir` resolves today, beside `slack.json` and `credentials.json`, whenever `XDG_CONFIG_HOME` is unset or absolute;
with a relative `XDG_CONFIG_HOME` the GitHub and Slack files stay under it while the machine file resolves under `$HOME/.config/lym` (D2).
It is JSON with the keys `name` (the machine's name) and `names` (every name the verb recorded for this machine, oldest first, the current one last).
A former name is a non-empty entry of `names` other than `name`.
Its writer's lock is `machine.lock` beside it, as the hub registry keeps `hubs.lock` beside `hubs.json`.

Rationale: the name must be one value for every hub on the machine and readable by a command run outside any hub;
JSON keeps `internal/machinename` on the standard library, as its leaf entry requires.
`names` keeps the current name as well, so a hand-broken `name` can be repaired back to it (D5).

Bound: the directory resolves from each process's environment, so a lym process started with another absolute `XDG_CONFIG_HOME` reads another file, as it already reads another Slack token.
`lym machine name` with no argument prints the file's path, so the operator can see which file a shell resolves.

### D2 — The per-user config directory resolvers live in `internal/standalonestate`

The resolver of the per-user config directory moves into `standalonestate.UserConfigDir` (new), beside `standalonestate.UserStateDir`.
It keeps `githubclient.CredentialDir`'s rule as it stands, a non-empty `XDG_CONFIG_HOME` taken as given, so the GitHub and Slack files keep their paths;
`githubclient.CredentialDir` delegates to it.
`standalonestate.AbsoluteUserConfigDir` (new) resolves the same directory but ignores a relative `XDG_CONFIG_HOME`, falling back to `$HOME/.config/lym`, and returns an error naming the variable for a relative `HOME` or `LOCALAPPDATA`, as `standalonestate.UserStateDir` treats the state variables.
The machine file resolves through `standalonestate.AbsoluteUserConfigDir` (new), so its path never depends on a process's working directory.
`internal/machinename` imports `internal/standalonestate` for it.

Rationale: `internal/machinename` is a leaf and cannot import `internal/githubclient`;
`internal/standalonestate` imports the standard library only, so the leaf stays closed to feature packages and the directory's rule lives in one package.
A relative `XDG_CONFIG_HOME` would let the verb write one file and the dobby and orch read another under their own working directories, which for the machine's identity is a wrong owner rather than a missing token.

### D3 — How `machinename.Name` resolves

- No resolvable config directory, since no variable D2 reads is set, or no file: the hostname rule, `machinename.FromHostname`, unchanged.
- A relative `HOME` or `LOCALAPPDATA`: the error `standalonestate.AbsoluteUserConfigDir` (new) returns, naming the variable; the way forward is setting it to an absolute path.
- A file with a valid `name`: that name.
- A file that does not read, does not decode, or holds an empty or invalid `name`: an error naming the file's path and the way forward, `lym machine name <name>` (which repairs it, D5) or removing the file.
  It never falls back to the hostname, since a wrong name silently changes which entries read as owned here.
- Unknown keys are ignored.
- The file is read on every call, with no cache.
- Every `machinename` error names its own way forward, the hostname case included.
  A caller that wraps an error of `machinename.Name` or `machinename.Names` (new) keeps that way forward and adds no hostname fix of its own.

Bound: a caller that reads the name per check, as every owner check does, takes a new name at its next check;
a process that resolved it once keeps the old name until it restarts.
The Slack relay is one: it fills its geometry once (`internal/hubgeom/slackgeom.go`), so it creates channels and signs posts under the old name until its restart.

Bound: the dobby's beam owner check (`internal/dobbyengine/loop.go`, `beamOwnedElsewhere`) reads a name error as not owned elsewhere and keeps the dobby going,
while batten's own owner check (`internal/battencli/owner.go`) refuses each pass with the name error;
a bad file therefore stalls the beam's passes rather than letting them run under a wrong owner.
The way forward is the one the error names.

`internal/machinename` stays the only production reader of the hostname and becomes the only reader of the machine file.
It exports what the verb and the landing match need:

- `machinename.FilePath` (new): the machine file's path and its lock's path, false when no directory resolves, and the error of a relative variable.
- `machinename.ReadFile` (new): the file at a path decoded into its `name` and `names` with no validation, and whether it exists.
- `machinename.Names` (new): the current name, resolved as `machinename.Name` does, and the former names.
  It drops every empty entry of `names`, since an empty owner reads as no owner and would match every entry without one.
  A non-empty entry that fails D4 stays, since it matches only an owner recorded under that exact string, which only this machine could have written, as under a hostname the hostname rule reduced before D4 existed.
- `machinename.CheckName` (new): the name rule (D4).

`internal/machinecli` (new) reads the file only through these and writes it through `internal/state` at the file and lock paths `machinename.FilePath` (new) gives.

### D4 — The name rule

A name set in the file is lower-case letters, digits and hyphens, starts and ends with a letter or digit, and is at most 63 characters.
An invalid name is refused, never rewritten.
The rule is one check in `internal/machinename`, used by both the reader (D3) and the verb (D5).
The hostname rule's output is not checked against it, so the rule refuses no machine's hostname-rule name.

Rationale: the name is written into board owners, into `agentname.FormatMachineAddress`'s `<machine>::<sender>` address and into `slackengine.ChannelName`;
excluding `.` and `:` keeps it from reading as a domain or a hub-qualified name, and the length is a DNS label's.

### D7 — A landing finished under a former name stays this machine's

`battencli.DobbyLandings` takes this machine's names, the current and the former, and counts a Land history entry as this machine's when its `Machine` is any of them.
Its caller's Landings seam (`internal/beamcli/dobbydeps.go`) reads them through `machinename.Names` (new) on each call.

Rationale: shed history entries keep the name they were written under; without this a landing finished just before a rename reads as another machine's.

### D8 — Tests reach the machine file and the hubs only through seams

`internal/machinename`'s own tests hand in the file's path through a seam.
`internal/machinecli` (new) takes the file's path, the hubs to sweep and the working directory through its deps, filled in production from `machinename.FilePath` (new), `hubregistry.Live` and the process's working directory.
Every test of `lym machine name` hands in a file path and hubs under its temporary directory, so no test writes the developer's machine file or a real hub's Bolt.
No test environment variable changes: tests of other packages that reach `machinename.Name` only read the developer's file, as lym itself does, and those that pin a name already inject it through their callers' seams.
On Windows `tmuxkit.Main` already points `LOCALAPPDATA`, and with it the config directory, at the test's temporary directory, so there those tests read no machine file.

Bound: off Windows, a developer whose machine file is bad sees those reading tests fail with the file's error and path; the way forward is the one D3 names.

Rationale: pointing `XDG_CONFIG_HOME` at a test directory for every package would also move git's, tmux's, gh's and the Slack and GitHub credential files for every test.

## Constraints

- `PATTERN-leaf-packages`: `internal/machinename` imports only the standard library and `internal/standalonestate` (D2), itself standard-library only.
  The entry's background gains `internal/standalonestate` in `internal/machinename`'s allowed set.
- `PATTERN-config-layers`: the machine file is no `configengine` module; it is machine identity beside the GitHub and Slack files, which also live outside `configengine`.
- `PATTERN-owner-machine`: owners are still written only by `boardengine`'s owner setter (D6), and the name still comes only from `internal/machinename`.
  The entry's background gains that the name comes from the machine file before the hostname.
  Its clause that the owner setter takes another machine's project only when `lym adopt` asks gains the rename sweep's replace set as the second case.
- `PATTERN-tolerant-records`: the file's reader ignores unknown keys and its writer rewrites it through `internal/state`.
- `PATTERN-lym-files`: the file and its lock are per-user state outside any hub, like `hubs.json` and `slack.json`; they name no `_lym` or `.lym` path, and the tracked-file lock rule does not apply.
- Slack, as the code stands: each orch's channel is recorded by id, so it keeps working under its old name after a rename; channels made after the relay's restart take the new name (`internal/slackengine/keeper.go`).
  After that restart, `retireEntry` compares a channel's name with its base name under the new machine name, so it archives an old channel under its undated old name.
  A later rename back to that old name makes the relay's new channel for that orch hit `name_taken`, and `adopt` refuses the archived channel;
  the way forward is the one that refusal names, renaming the archived channel in Slack, which frees the name.
- Every owner comparison is string equality: `boardengine.OwnedElsewhere`, which the pair teardown seam reads through its owner reader, and the dobby's own owner comparisons. So once a hub is swept every reader agrees, and an unswept owner under the old name reads as another machine's.

## Auto-mode assumptions

- D1, D3, D4, D5, D6 and D8 were picked without an operator; the support log's question ledger holds each pick.

## Open risks

## Acceptance criteria

- With no machine file, `machinename.Name` returns the hostname rule's name (D1, D3).
- With a file holding the name `osl46`, `machinename.Name` returns `osl46` whatever the hostname (D1, D3).
- A file that does not decode, or holds an empty or invalid name, makes `machinename.Name` return an error naming the file's path; it never returns the hostname name (D3, D4).
- A file with an unknown key reads its name (D3).
- `machinename.Names` (new) returns the file's name and every other non-empty entry of its `names`, one that fails D4 included (D3, D7).
- A refusal for a bad machine file, a `lym batten run` refusal included, names the file's path and `lym machine name`, and no hostname fix (D3).
- The name check accepts `osl46` and `neve` and refuses `OSL46`, `osl.46`, `osl:46`, `osl_46`, a leading or trailing hyphen and a 64-character name (D4).
- `githubclient.CredentialDir` and `standalonestate.UserConfigDir` (new) return the same directory for the same environment, a relative `XDG_CONFIG_HOME` included, on Linux and on Windows's variables (D2).
- `standalonestate.AbsoluteUserConfigDir` (new) returns `$HOME/.config/lym` for a relative `XDG_CONFIG_HOME`, and an error naming the variable for a relative `HOME` or `LOCALAPPDATA`, which `machinename.Name` returns instead of the hostname name (D2, D3).
- `internal/machinename`'s import allowlist holds only the standard library and `internal/standalonestate` (D2).
- `battencli.DobbyLandings` lists a run whose Land finished under a former name (D7).
- `internal/machinename`'s tests read the machine file only at the path their seam hands in (D8).
- Review: the doc of `internal/machinename` describes the machine file and its precedence over the hostname rule (D1, D3).

## PATTERN changes

- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set.
