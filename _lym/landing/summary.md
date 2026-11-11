# machinename: read the machine name from machine.json first

`machinename.Name` now reads `machine.json` from the per-user lym config directory before it applies the hostname rule.
The name is then one value for every hub on the machine, and an operator can set it without renaming the host.
This change holds the reader side only.
The `lym machine name` verb that writes the file (`internal/machinecli`) is not in this diff, though the error messages already point to it.

## What changes

- With no file, or no resolvable config directory, `machinename.Name` returns the hostname rule's name as before.
- With a file, it returns the file's `name`.
  A file that does not read, does not decode, or holds an empty or invalid `name` is an error naming the file's path and its way forward.
  It never falls back to the hostname, since a wrong name silently changes which entries read as owned here.
- The file is JSON with `name` and `names` (every name recorded for the machine, oldest first, the current one last).
  Unknown keys are ignored, and the file is read on every call.
- New exports in `internal/machinename`: `FilePath` (file and lock paths), `ReadFile` (decode with no validation), `Names` (current and former names), `CheckName` (the name rule).
- The name rule: lower-case letters, digits and hyphens, starting and ending with a letter or digit, at most 63 characters.
  The hostname rule's output is not checked against it, so no machine's hostname-rule name is refused.
- Every `machinename` error names its own way forward, the hostname case included.
  The wrappers in `internal/battencli` (`thisMachine`, the run's machine stamp) and `internal/beamcli` (`machineLandings`) keep that way forward and add no hostname fix of their own.
- `battencli.DobbyLandings` takes this machine's names and counts a Land history entry as this machine's when its `Machine` is any of them.
  A landing finished just before a rename therefore still counts.
  `internal/beamcli/dobbydeps.go` reads the names through `machinename.Names` on each call.

## Design decisions

- The per-user config directory resolver moves into `standalonestate.UserConfigDir`, and `githubclient.CredentialDir` delegates to it.
  It keeps the old rule, including a relative `XDG_CONFIG_HOME` taken as given, so the GitHub and Slack files keep their paths.
- `standalonestate.AbsoluteUserConfigDir` is new and ignores a relative `XDG_CONFIG_HOME`, falling back to `$HOME/.config/lym`.
  A relative `HOME` or `LOCALAPPDATA` is an error naming the variable.
  The machine file resolves through it, so its path never depends on a process's working directory.
  With a relative `XDG_CONFIG_HOME` the machine file therefore sits in a different directory from the Slack and GitHub files.
- `internal/machinename` now imports `internal/standalonestate`, which imports only the standard library.
  `PATTERN-leaf-packages` and `PATTERN-owner-machine` are updated to say so, and `docs/overview.md` matches.
- Non-empty former names that fail the name rule stay in `Names`, since they match only an owner this machine itself recorded.
  Empty entries are dropped, since an empty owner reads as no owner.

## Behaviour to know

- A process that resolved the name once keeps it until restart.
  The Slack relay fills its geometry once, so it keeps the old name for channels and posts until it restarts.
- The dobby's beam owner check reads a name error as not owned elsewhere, while batten's owner check refuses each pass with the error.
  A bad file therefore stalls the beam's passes rather than running them under a wrong owner.
- Off Windows, a developer whose machine file is bad sees tests that reach `machinename.Name` fail with the file's error and path.
  The `internal/machinename` tests hand in the path through a seam and read no developer file.

## To check by hand

- On a machine with a real `machine.json` containing a name other than the hostname, run a command that reads the owner (for example `lym batten run`) and confirm it takes the file's name.
- Write a broken `machine.json` and confirm the refusal names the file's path and `lym machine name`, with no hostname advice.
