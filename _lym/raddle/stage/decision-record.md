# Decision record — machine name from a machine-wide config

## Goal

Let the operator give a machine a short name of its own in a machine-wide file lym reads, such as `osl46` for the host `osl-1046`, so the name is no longer only the hostname rule.
A machine with no name in the file keeps the hostname rule and changes no behaviour,
except that a relative `HOME`, or `LOCALAPPDATA` on Windows, now makes its name an error until the variable is set to an absolute path (D3).
A machine that takes a new name keeps every owner it already recorded under the old one.

## Scope

In:

- The machine file, its reader in `internal/machinename` and the name rule it enforces.
- The per-user config directory resolvers in `internal/standalonestate`, used by `internal/machinename` and `internal/githubclient`.
- A new `lym machine name` verb that shows the name and sets it.
- The owner sweep the verb runs over every hub of the machine, through `boardengine.SetOwners`; it is what the entry's open migration question needs, so it reaches past `internal/machinename`.
- The landing-history match in `battencli.DobbyLandings`, which counts former names as this machine.

Out:

- Slack channel names: the following Slack task reorders them; this task changes no Slack code (see Constraints for what a rename does there).
- The cross-machine bridge `slack-machine-bridge` and its `<machine>::<sender>` routing.
- Any hostname rule change for a machine without a file.

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

### D5 — The `lym machine name` verb

A new module `machine`, package `internal/machinecli` (new), wired as a cobra subtree under the root per `PATTERN-cli-cobra`.
`lym machine name` runs from any directory, inside a hub or not.

- With no argument it prints the name, its source (`file` or `hostname`), the file's path and the former names, and writes nothing.
  A bad file prints the file's error and path instead of a name.
- With a name it validates it against D4, refusing an invalid one with the rule as the way forward.
- With no resolvable config directory, or a relative variable, it refuses to write, naming the variables D2 reads.
- It pulls the Bolt of every hub it will sweep (D6), through `fabricengine.Bolt.Pull`, then reads each one's owners through `boardengine.ReadOwners`.
  A hub whose pull returns an error or skips its fetch is checked against its local Bolt, and the output names it as checked locally only.
- It takes the hostname rule's name, `machinename.FromHostname`, as one of this machine's names beside the current and former ones, over a bad file too; when the hostname yields no name, it adds none.
- It refuses a name that one of those boards records as an owner and that is not one of this machine's names, naming that board and owner: that name belongs to another machine.
- It then writes the file through `internal/state`: `name` becomes the new name;
  the old name and the hostname rule's name are each appended to `names` when absent from it;
  every empty entry is dropped, and the output says so;
  the new name is then moved to the end of `names`.
- It then runs the owner sweep (D6).
- A run with the name already set writes the file only when that rewrite changes `names`, and runs the sweep again, so a re-run finishes a sweep a hub failed.
- Over a bad file it reads the file itself through `machinename.ReadFile` (new) instead of through `machinename.Name`, so it repairs what D3 refuses:
  a file that decodes keeps its `names`, rewritten as the write step says, and drops its invalid `name`, which no owner can hold since `machinename.Name` refused it;
  the last valid name stays in `names`, so a repair back to it passes the collision check.
  A file that does not decode is replaced and its names are lost, so `names` holds only the hostname rule's name and the new one.
  The output says so.
  A repair back to a lost name that the boards hold is refused by the collision check;
  its way forward, which the refusal names, is writing `{"name": "<name>"}` to the file's path by hand, which `machinename.Name` then reads as the name.
  An owner left under a lost name after a rename to another name is taken back with `lym adopt`.
  A landing finished under a lost name no longer reads as this machine's (D7), so one the dobby has not yet counted counts toward no board tidy (`internal/dobbyengine/tidy.go`, `countLandings`);
  later landings count as before, and restoring the lost name in `names` by hand makes such a landing count.

Bound: the verb writes only this machine's file and owners recorded under this machine's former names; it never writes an owner recorded under any other name.
Bound: an owner another machine pushes to origin between the verb's pull and its file write, and an owner on a hub whose pull failed that only origin holds, are not seen by the collision check; the replace set (D6) still keeps the write off them.
Bound: a former name is taken to stay this machine's, so a former name another host has since taken as its own, by hostname or file, passes the collision check.
A name another live machine holds, by hostname or file, but under which it has recorded no owner on any swept board, passes it too.
So does the hostname rule's name when another host shares it, as two hosts whose hostnames differ only after the first dot do; this machine's sweep then moves that host's open entries under this machine.
A name shared with another machine, by any of these cases or the one above, makes each machine read the other's owned entries as its own, since every owner comparison is string equality (Constraints), so either can start, resume and land the other's runs;
a re-run also sweeps that host's owners and D7 counts its landings.
The way forward is renaming, with `lym machine name` on it, the machine that holds the shared name as its current name, or either one when both do, to a name no other machine holds.
A sweep whose replace set holds the shared name, the renamed machine's own or an earlier one of a machine holding it as a former name, moves the other machine's open entries under that sweep's machine;
each machine then takes back with `lym adopt` every entry of its own that the other's sweep moved.

### D6 — The owner sweep keeps owners across a rename

The verb sweeps every hub `hubregistry.Live` lists, plus the hub of the working directory when it lies in one.
For each hub it makes one `boardengine.SetOwners` call with the new name and the replace set, this machine's former names, which pulls, commits and pushes the Bolt as an owner write does today.
`boardengine.OwnerRequest` gains a field naming the replace set.
With it set, the write itself selects, after its pull, every open beam project and the prime whose recorded owner is in the set, and rewrites each to the request's machine, though that owner reads as another machine;
it writes no other owner, and `Takeover` stays unset.
A done entry keeps its recorded owner, since the owner write never writes one.

- A hub whose write fails, as on an unreachable origin or a refused push, is reported with its error and the way forward, a re-run of `lym machine name <name>`; the other hubs go on and the verb exits non-zero.
- A hub whose write returns entries in `OwnerWrite.Lost`, as when a competing origin write kept them, is reported per entry with the owner that stands and the way forward, a re-run of `lym machine name <name>`, which takes back an entry whose standing owner is a former name; the verb exits non-zero.
  An entry whose standing owner is another name stays that machine's, and `lym adopt` takes it back when it should be this machine's.
- A hub whose write records nothing is reported as having nothing to sweep.
  The verb reads before the write whether the hub's Bolt is ahead of origin, so a re-run whose write only pushes a commit an earlier run left unpushed reports that hub as pushed.

Bound: a hub neither in the registry nor holding the working directory is not swept; the way forward is running the verb from inside it.

Bound: between the file write and its hub's sweep, every owner check in that hub reads the hub's owners as another machine's.
No pause holds every such check: the orch's bring-up pauses each entry it reads as owned elsewhere, and a hub pause only delays its reconcile until the lift (`internal/orchcli/doc.go`, the owner pause at bring-up);
the dobby holds a run only under an own-scope entry carrying `pauserecord.HoldCaller` (`internal/dobbyengine/passes.go`, `Held`).
The verb's output names each recovery:

- a run that takes an owner pause through `tendercli.PauseOwnedElsewhere` or a hibernate wake, from the dobby's probe, a pass's own owner check, the orch's bring-up or loom's pre-push among others: once the pause stands, `tendercli.EndOwnedElsewhere` ends the run's processes on this machine, except a Finalize past its parent-side merge (`internal/tendercli/doc.go`, Owned elsewhere; `internal/hibernateengine/doc.go`).
  After the hub's sweep, the dobby's probe reads the owner as this machine, lifts its own owner pause and runs a full pass, which drives the run again (`internal/dobbyengine/doc.go`).
  A pause the dobby's probe does not lift is lifted with `lym tender resume <slug>`, and the dobby gives the run a full pass at that lift;
  a resume alone restarts nothing, since its resume lines reach only live sessions (`internal/tenderengine/doc.go`, Resume).
  A hub whose sweep committed but did not push holds the new owner in its local Bolt only:
  the dobby's probe, which reads the local Bolt, lifts its owner pause and the resume, judged against origin, re-takes it, at each probe until a re-run of `lym machine name <name>` pushes the commit.
  The verb's output for that hub names that re-run as the recovery.
- a Finalize halted by its owner check: `lym loom resume` in the run's pair;
- a dobby or orch that skipped or refused a worktree as owned elsewhere: `lym beam start <slug>`, or `lym beam start root` for the prime.

Lift any pause with `lym tender resume` only after that hub's sweep reports done:
a lift skips the deferred push of every pair it reads as owned by another machine (`internal/tenderengine/doc.go`, Resume), so a lift before the sweep leaves those pairs unpushed.

Rationale: rewriting owners to the new name keeps one name on the board for every machine and for the Slack and cross-machine work that follows.
Selecting the owners under the write's own pull sweeps an owner only origin holds and pulls every hub, even one whose local Bolt holds no former-name owner.

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

- None beyond the bounds the Decisions state.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - error-wrappers-contradict-way-forward (class design, severity MEDIUM, open in rounds 2)
  - former-names-unvalidated (class design, severity MEDIUM, open in rounds 2)
  - hostname-name-not-in-names (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- With no machine file, `machinename.Name` returns the hostname rule's name (D1, D3).
- With a file holding the name `osl46`, `machinename.Name` returns `osl46` whatever the hostname (D1, D3).
- A file that does not decode, or holds an empty or invalid name, makes `machinename.Name` return an error naming the file's path; it never returns the hostname name (D3, D4).
- A file with an unknown key reads its name (D3).
- `machinename.Names` (new) returns the file's name and every other non-empty entry of its `names`, one that fails D4 included (D3, D7).
- An empty `names` entry is neither a former name for D6's replace set nor a name D7 matches (D3, D6, D7).
- A refusal for a bad machine file, a `lym batten run` refusal included, names the file's path and `lym machine name`, and no hostname fix (D3).
- The name check accepts `osl46` and `neve` and refuses `OSL46`, `osl.46`, `osl:46`, `osl_46`, a leading or trailing hyphen and a 64-character name (D4).
- `githubclient.CredentialDir` and `standalonestate.UserConfigDir` (new) return the same directory for the same environment, a relative `XDG_CONFIG_HOME` included, on Linux and on Windows's variables (D2).
- `standalonestate.AbsoluteUserConfigDir` (new) returns `$HOME/.config/lym` for a relative `XDG_CONFIG_HOME`, and an error naming the variable for a relative `HOME` or `LOCALAPPDATA`, which `machinename.Name` returns instead of the hostname name (D2, D3).
- `internal/machinename`'s import allowlist holds only the standard library and `internal/standalonestate` (D2).
- `lym machine name` with no argument prints the name, its source, the file path and the former names, and writes nothing (D5).
- `lym machine name osl46` on a machine named `osl-1046` writes the name `osl46` with `names` holding `osl-1046` then `osl46` (D5).
- `lym machine name` refuses an invalid name and writes nothing (D4, D5).
- `lym machine name neve` is refused when a swept board records `neve` as an owner that is not one of this machine's names, an owner only origin held before the verb's pull included, and writes nothing (D5).
- `lym machine name osl46` over a file holding an invalid name keeps the file's non-empty `names` entries, adds the hostname rule's name when absent and writes the new name; over a file that does not decode it writes the new name with `names` holding the hostname rule's name then `osl46` (D5).
- `lym machine name osl46` over a file whose `name` was hand-broken and whose `names` ends in `osl46` succeeds while swept boards record `osl46` as an owner (D5).
- `lym machine name osl46` over a hand-written file holding only the name `osl46` records the hostname rule's name in `names` and sweeps owners recorded under it (D5, D6).
- `lym machine name` over a file whose `names` holds an empty entry drops it from `names` and says so in its output (D5).
- After a rename, every open beam project's owner and the prime owner equal to a former name on each swept hub reads the new name, an owner only origin held before the sweep included; a done entry and an owner recorded under another name are untouched (D6).
- An owner write with the replace set and no `Takeover` rewrites every open beam project and the prime whose owner, read after its pull, is in the set, and leaves every other owner (D6).
- A hub whose sweep fails is reported with the re-run as its way forward while the other hubs are swept, and a re-run with the same name sweeps it (D5, D6).
- `battencli.DobbyLandings` lists a run whose Land finished under a former name (D7).
- `internal/machinename`'s tests read the machine file only at the path their seam hands in (D8).
- `lym machine name`'s tests write the machine file only at the path their seam hands in, and sweep only hubs under their temporary directory (D8).
- Review: the doc of `internal/machinename` describes the machine file and its precedence over the hostname rule (D1, D3).
- Review: the verb's output after a rename names the recoveries D6's window bound lists and advises lifting a pause only after the hub's sweep reports done (D6).

## PATTERN changes

- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set.
- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`.

## Notes for the plan writer

- Both PATTERN changes are background-only; their overview lines stay as they are.
- The sole-source scan in `internal/machinename/machinename_test.go` is the natural place to also pin that no production file outside `internal/machinename` names the machine file.
