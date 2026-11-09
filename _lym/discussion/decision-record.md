# Decision record — machine name from a machine-wide config

## Goal

Let the operator give a machine a short name of its own in a machine-wide file lym reads, such as `osl46` for the host `osl-1046`, so the name is no longer only the hostname rule.
A machine with no name in the file keeps the hostname rule and changes no behaviour.
A machine that takes a new name keeps every owner it already recorded under the old one.

## Scope

In:

- The machine file, its reader in `internal/machinename` and the name rule it enforces.
- A per-user config directory resolver shared by `internal/machinename` and `internal/githubclient`.
- A new `lym machine name` verb that shows the name and sets it.
- The owner sweep the verb runs over every hub of the machine, through `boardengine.SetOwners`; it is what the entry's open migration question needs, so it reaches past `internal/machinename`.
- The landing-history match in `battencli.DobbyLandings`, which counts former names as this machine.

Out:

- Slack channel names: the following Slack task reorders them; this task changes no Slack code (see Constraints for why a rename is safe there).
- The cross-machine bridge `slack-machine-bridge` and its `<machine>::<sender>` routing.
- Any hostname rule change for a machine without a file.

## Decisions

### D1 — The machine file lives in the per-user config directory

The file is `machine.json` in the per-user lym config directory: `$XDG_CONFIG_HOME/lym`, else `$HOME/.config/lym`, and `%LOCALAPPDATA%\lym` on Windows.
That is the directory `githubclient.CredentialDir` resolves today, beside `slack.json` and `credentials.json`.
It is JSON with the keys `name` (the machine's name) and `former_names` (names this machine went by before, newest last).
Its writer's lock is `machine.lock` beside it, as the hub registry keeps `hubs.lock` beside `hubs.json`.

Rationale: the name must be one value for every hub on the machine and readable by a command run outside any hub;
JSON keeps `internal/machinename` on the standard library, as its leaf entry requires.

Bound: the directory resolves from each process's environment, so a lym process started with another `XDG_CONFIG_HOME` reads another file, as it already reads another Slack token.
`lym machine name` with no argument prints the file's path, so the operator can see which file a shell resolves.

### D2 — One per-user config directory resolver in `internal/standalonestate`

The resolver of the per-user config directory moves into `standalonestate.UserConfigDir` (new), beside `standalonestate.UserStateDir`.
It keeps `githubclient.CredentialDir`'s rule as it stands, a non-empty `XDG_CONFIG_HOME` taken as given, so the GitHub and Slack files keep their paths;
`githubclient.CredentialDir` delegates to it.
`internal/machinename` imports `internal/standalonestate` for it.

Rationale: `internal/machinename` is a leaf and cannot import `internal/githubclient`;
`internal/standalonestate` imports the standard library only, so the leaf stays closed to feature packages and the directory has one resolver.

### D3 — How `machinename.Name` resolves

- No resolvable config directory, or no file: the hostname rule, `machinename.FromHostname`, unchanged.
- A file with a valid `name`: that name.
- A file that does not read, does not decode, or holds an empty or invalid `name`: an error naming the file's path and the way forward, `lym machine name <name>` (which repairs it, D5) or removing the file.
  It never falls back to the hostname, since a wrong name silently changes which entries read as owned here.
- Unknown keys are ignored.
- The file is read on every call, with no cache.

Bound: a caller that reads the name per check, as every owner check does, takes a new name at its next check;
a process that resolved it once keeps the old name until it restarts.
The Slack relay is one: it fills its geometry once (`internal/hubgeom/slackgeom.go`), so it creates channels and signs posts under the old name until its restart.

`internal/machinename` stays the only production reader of the hostname and becomes the only reader of the machine file.

### D4 — The name rule

A name set in the file is lower-case letters, digits and hyphens, starts and ends with a letter or digit, and is at most 63 characters.
An invalid name is refused, never rewritten.
The rule is one check in `internal/machinename`, used by both the reader (D3) and the verb (D5).
The hostname rule's output is not checked against it, so no machine without a file changes behaviour.

Rationale: the name is written into board owners, into `agentname.FormatMachineAddress`'s `<machine>::<sender>` address and into `slackengine.ChannelName`;
excluding `.` and `:` keeps it from reading as a domain or a hub-qualified name, and the length is a DNS label's.

### D5 — The `lym machine name` verb

A new module `machine`, package `internal/machinecli` (new), wired as a cobra subtree under the root per `PATTERN-cli-cobra`.
`lym machine name` runs from any directory, inside a hub or not.

- With no argument it prints the name, its source (`file` or `hostname`), the file's path and the former names, and writes nothing.
  A bad file prints the file's error and path instead of a name.
- With a name it validates it against D4, refusing an invalid one with the rule as the way forward.
- It refuses a name that a board it will sweep (D6) records as an owner and that is not this machine's current or former name, naming that board and owner: that name belongs to another machine.
- It then writes the file through `internal/state`: `name` becomes the new name;
  the old name, when it differs, is appended to `former_names`;
  the new name is removed from `former_names`.
- It then runs the owner sweep (D6).
- A run with the name already set writes nothing and runs the sweep again, so a re-run finishes a sweep a hub failed.
- Over a bad file it reads the file itself instead of through `machinename.Name`, so it repairs what D3 refuses:
  a file that decodes keeps its `former_names` and drops its invalid `name`, which no owner can hold since `machinename.Name` refused it;
  a file that does not decode is replaced, its former names are lost, and the old name recorded is the hostname rule's.
  The output says so; an owner left under a lost name is taken back with `lym adopt`.

Bound: the verb writes only this machine's file and owners recorded under this machine's former names; it never writes an owner recorded under any other name.
Bound: the collision check reads each hub's local Bolt before the sweep's pull, so an owner only origin holds is not seen; the replace set (D6) still keeps the write off it.
Bound: a former name is taken to stay this machine's.
A host that later takes a former name as its own, by hostname or file, has its owners swept by a re-run and its landings counted by D7; the way forward is giving that host a distinct name.

### D6 — The owner sweep keeps owners across a rename

The verb sweeps every hub `hubregistry.Live` lists, plus the hub of the working directory when it lies in one.
For each hub it reads the Bolt's owners through `boardengine.ReadOwners`.
Every beam project's owner and the prime owner equal to a former name is rewritten to the new name in one `boardengine.SetOwners` call, which pulls, commits and pushes the Bolt as an owner write does today.
`boardengine.OwnerRequest` gains a field naming the owners the write may replace.
With it set, the write overwrites an owner in that set, though the owner reads as another machine, and leaves any other owner, reporting it in `Lost`; `Takeover` stays unset.

- A hub whose write fails, as on an unreachable origin or a refused push, is reported with its error and the way forward, a re-run of `lym machine name <name>`; the other hubs go on and the verb exits non-zero.
- A hub with no owner under a former name is reported as having nothing to sweep.

Bound: a hub neither in the registry nor holding the working directory is not swept; the way forward is running the verb from inside it.

Bound: between the file write and its hub's sweep, every owner check in that hub reads the hub's owners as another machine's.
The verb's output names each recovery, and advises a hub pause (`lym tender pause`) before a rename for an operator who wants none to arise:

- a run that takes an owner pause: the dobby's owner-pause lift, or `lym tender resume <slug>`;
- a Finalize halted by its owner check: `lym loom resume` in the run's pair;
- a dobby or orch that skipped or refused a worktree as owned elsewhere: `lym beam start <slug>`, or `lym beam start root` for the prime;
- a hibernate wake that ended an owned-elsewhere run's processes: `lym tender resume <slug>`.

Rationale: rewriting owners to the new name keeps one name on the board for every machine and for the Slack and cross-machine work that follows.

### D7 — A landing finished under a former name stays this machine's

`battencli.DobbyLandings` counts a Land history entry as this machine's when its `Machine` is the current name or a former name.

Rationale: shed history entries keep the name they were written under; without this a landing finished just before a rename reads as another machine's.

### D8 — `internal/machinename`'s tests reach the file through a seam

The package's own tests hand in the file's path through a seam, so they never read the developer's file.
No test environment variable changes: tests outside the package that reach `machinename.Name` read the developer's file as lym itself does, and those that pin a name already inject it through their callers' seams.

Rationale: pointing `XDG_CONFIG_HOME` at a test directory for every package would also move git's, tmux's, gh's and the Slack and GitHub credential files for every test.

## Constraints

- `PATTERN-leaf-packages`: `internal/machinename` imports only the standard library and `internal/standalonestate` (D2), itself standard-library only.
- `PATTERN-config-layers`: the machine file is no `configengine` module; it is machine identity beside the GitHub and Slack files, which also live outside `configengine`.
- `PATTERN-owner-machine`: owners are still written only by `boardengine`'s owner setter (D6), and the name still comes only from `internal/machinename`.
- `PATTERN-tolerant-records`: the file's reader ignores unknown keys and its writer rewrites it through `internal/state`.
- `PATTERN-lym-files`: the file and its lock are per-user state outside any hub, like `hubs.json` and `slack.json`; they name no `_lym` or `.lym` path, and the tracked-file lock rule does not apply.
- Slack, as the code stands: each orch's channel is recorded by id, so it keeps working under its old name after a rename and is archived under that name; channels made after the relay's restart take the new name (`internal/slackengine/keeper.go`, `retireEntry`).
- Every owner comparison is string equality: `boardengine.OwnedElsewhere`, which the pair teardown seam reads through its owner reader, and the dobby's own owner comparisons. So once a hub is swept every reader agrees, and an unswept owner under the old name reads as another machine's.

## Auto-mode assumptions

- D1, D3, D5 and D6 were picked without an operator; the support log's question ledger holds each pick.

## Open risks

- None beyond the bounds the Decisions state.

## Acceptance criteria

- With no machine file, `machinename.Name` returns the hostname rule's name (D1, D3).
- With a file holding the name `osl46`, `machinename.Name` returns `osl46` whatever the hostname (D1, D3).
- A file that does not decode, or holds an empty or invalid name, makes `machinename.Name` return an error naming the file's path; it never returns the hostname name (D3, D4).
- A file with an unknown key reads its name (D3).
- The name check accepts `osl46` and `neve` and refuses `OSL46`, `osl.46`, `osl:46`, `osl_46`, a leading or trailing hyphen and a 64-character name (D4).
- `githubclient.CredentialDir` and `standalonestate.UserConfigDir` (new) return the same directory for the same environment, a relative `XDG_CONFIG_HOME` included, on Linux and on Windows's variables (D2).
- `internal/machinename`'s import allowlist holds only the standard library and `internal/standalonestate` (D2).
- `lym machine name` with no argument prints the name, its source, the file path and the former names, and writes nothing (D5).
- `lym machine name osl46` on a machine named `osl-1046` writes the name `osl46` with `osl-1046` in `former_names` (D5).
- `lym machine name` refuses an invalid name and writes nothing (D4, D5).
- `lym machine name neve` is refused when a swept board records `neve` as an owner that is not this machine's former name, and writes nothing (D5).
- `lym machine name osl46` over a file holding an invalid name keeps the file's `former_names` and writes the new name; over a file that does not decode it writes the new name with the hostname rule's name as the former name (D5).
- After a rename, every beam owner and the prime owner equal to the old name on each swept hub reads the new name, and an owner recorded under another name is untouched (D6).
- An owner write with the replace set and no `Takeover` overwrites an owner in the set and leaves an owner outside it, reporting that one in `Lost` (D6).
- A hub whose sweep fails is reported with the re-run as its way forward while the other hubs are swept, and a re-run with the same name sweeps it (D5, D6).
- `battencli.DobbyLandings` lists a run whose Land finished under a former name (D7).
- `internal/machinename`'s tests read the machine file only at the path their seam hands in (D8).
- Review: the doc of `internal/machinename` describes the machine file and its precedence over the hostname rule (D1, D3).
- Review: the verb's output after a rename names the recoveries D6's window bound lists (D6).

## PATTERN changes

- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set.
- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`.

## Notes for the plan writer

- Both PATTERN changes are background-only; their overview lines stay as they are.
- `PATTERN-leaf-packages`' background gains `internal/standalonestate` in `internal/machinename`'s allowed set.
- `PATTERN-owner-machine`'s background gains that the name comes from the machine file before the hostname.
  Its clause that the owner setter takes another machine's project only when `lym adopt` asks gains the rename sweep's replace set as the second case.
- The sole-source scan in `internal/machinename/machinename_test.go` is the natural place to also pin that no other production file names the machine file.
