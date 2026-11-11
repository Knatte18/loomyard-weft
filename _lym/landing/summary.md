# Read the machine name from machine.json before the hostname rule

This machine's name now comes from `machine.json` in the per-user lym config directory, when that file exists, and from the hostname rule otherwise.
One name then holds for every hub on the machine, and a command run outside any hub reads it too.
The verb that writes the file, `lym machine name`, is the follow-up task `machine-name-verb`.

## What changes

- **`internal/standalonestate`** gains the per-user config directory resolvers.
  `UserConfigDir` keeps the rule `githubclient.CredentialDir` used, a non-empty `XDG_CONFIG_HOME` taken as given, so the GitHub and Slack files keep their paths.
  `AbsoluteUserConfigDir` ignores a relative `XDG_CONFIG_HOME` in favour of `$HOME/.config/lym`,
  and returns an error naming the variable for a relative `HOME` or `LOCALAPPDATA`.
  `errRelativeHome`'s text now names `HOME`.
- **`internal/githubclient`**: `CredentialDir` delegates to `standalonestate.UserConfigDir`, and the leaf allowlist admits `internal/standalonestate`.
- **`internal/machinename`**:
  - `Name` reads the file through `AbsoluteUserConfigDir` on every call.
    With no file, or no directory to hold one, it falls back to the unchanged hostname rule.
    A file that does not read, does not decode, or holds an empty or invalid `name` is an error naming the file's path and the way forward:
    `lym machine name <name>` or removing the file.
    Such a file never falls back to the hostname, because a wrong name silently changes which entries read as owned here.
    Unknown keys are ignored.
  - New exports: `FilePath` (file and lock paths), `ReadFile` (decode with no validation), `Names` (current plus former names)
    and `CheckName` (lower-case letters, digits and hyphens, a letter or digit at each end, at most 63 characters).
  - `Names` drops empty entries of `names`, since an empty owner would match every unowned entry.
    It keeps a non-empty entry that fails the name rule, since only this machine could have written an owner under it.
  - Every error the package returns names its own way forward, the hostname-read error included.
- **`internal/battencli`**: `DobbyLandings` takes this machine's names and counts a Land finished under any of them,
  so a landing finished just before a rename stays this machine's.
  The owner check and the shed build no longer append a hostname fix to a name error; they keep the error's own way forward.
  The owner check's empty-name branch is gone, since `machinename.Name` never returns an empty name without an error.
- **`internal/beamcli`**: the dobby's Landings seam reads `machinename.Names` on each call and drops its hostname fix.
- **`internal/boardengine`**: the owner setter's no-machine refusal no longer says the name fails only on the hostname.
- `PATTERN-leaf-packages` admits `internal/standalonestate` into `internal/machinename`'s import set;
  `PATTERN-owner-machine` states that the name comes from the machine file before the hostname rule.

## Design decisions

- `internal/machinename`'s tests reach the file only through an unexported `resolveNames` seam that takes the file's locator and the hostname reader.
  No test reads or writes the developer's machine file.
  Tests of other packages that call `machinename.Name` read the developer's real file, as lym does (decision D8).
- The refusals name `lym machine name`, which the follow-up task `machine-name-verb` adds.
  Until it lands, removing the file is the working way forward the same refusals name.
- The `PATTERN-owner-machine` clause on the rename sweep's replace set is left to `machine-name-verb`, which adds the sweep.

## Testing

- `machinename`: a table over `resolveNames` covering no directory, no file, a valid name over the hostname, an unknown key, former names,
  undecodable, empty and invalid names, a relative variable and an unreadable hostname; an unreadable-file case; a `CheckName` table; `FilePath`'s names; the allowlist now admitting `standalonestate`.
- `standalonestate`: a table over both config-directory rules on both platform rows.
- `githubclient`: `CredentialDir` matches `UserConfigDir` under a relative `XDG_CONFIG_HOME`.
- `battencli` and `beamcli`: the name-error rows keep the error's way forward and add no hostname fix,
  and `DobbyLandings` lists a run landed under a former name.
