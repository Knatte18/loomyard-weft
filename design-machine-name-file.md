# machinename reads the machine name from machine.json before the hostname rule

`machine-name-file` · Task · feature, config, slack · draft

Decision record: [Sketch](decision-machine-name-file.md)

Depends on: [`adopt-here-archives`](design-adopt-here-archives.md)

Filed from the Sketch `machine-name-config`.

Takes: the decision record's D1-D4, D7 and D8 for internal/machinename's tests
Delivers: machinename.Name reading machine.json from the absolute per-user config directory before the hostname rule, with every name error naming its own way forward; githubclient.CredentialDir delegating to standalonestate.UserConfigDir; battencli.DobbyLandings counting landings under former names; the name-error wrappers in internal/battencli and internal/beamcli rid of their hostname fix
Exports: standalonestate.UserConfigDir, standalonestate.AbsoluteUserConfigDir, machinename.FilePath, machinename.ReadFile, machinename.Names, machinename.CheckName, battencli.DobbyLandings taking this machine's names

The part changes these packages:

- internal/standalonestate (about 100 changed lines)
- internal/githubclient (about 15 changed lines)
- internal/machinename (about 180 changed lines)
- internal/battencli (about 30 changed lines)
- internal/beamcli (about 50 changed lines)
